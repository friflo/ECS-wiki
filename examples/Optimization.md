
Examples in this section are targeting for performance optimization.  
The same functionality can be realized by using the features described in the general examples.  
Performance optimizations are achieved by SIMD, multi threading / parallelization, batching or bulk operations.

Optimization examples are part of the unit tests see:
[Tests/ECS/Examples](https://github.com/friflo/Friflo.Engine.ECS/tree/main/src/Tests/ECS/Examples)

<br/>


# Entity Queries

## Enumerate Query Chunks

**Optimization**: Replace a `query.ForEachEntity(( ... ) => lambda)` by two `foreach` loops.  
This approach avoids the more expensive lambda calls.

Also as described in the intro enumeration of a query result is fundamental for an ECS.  
Components are returned as [Chunk](https://github.com/friflo/Friflo.Engine-docs/blob/main/api/Chunk_T_.md)'s and are suitable for
[Vectorization - SIMD](https://en.wikipedia.org/wiki/Single_instruction,_multiple_data).

```csharp
public static void EnumerateQueryChunks()
{
    var store   = new EntityStore();
    for (int n = 0; n < 3; n++) {
        store.CreateEntity(new MyComponent{ value = n + 42 });
    }
    var query = store.Query<MyComponent>();
    foreach (var (components, entities) in query.Chunks)
    {
        for (int n = 0; n < entities.Length; n++) {
            Console.WriteLine(components[n].value);                  // > 42  43  44
        }
    }
    // Caution! This alternative to iterate components is much slower
    foreach (var entity in query.Entities) {
        Console.WriteLine(entity.GetComponent<MyComponent>().value); // > 42  43  44
    }
}
```
🔥 **Update** - Added note for slower iteration alternative.  
The alternative should be used only for small result set with less than 10 entities.

<br/>


## Parallel Query Job

**Optimization**: Execute a `query` on multiple CPU cores in parallel.

To minimize execution time for large queries a [QueryJob](https://github.com/friflo/Friflo.Engine-docs/blob/main/api/QueryJob.md) can be used.  
It provides the same functionality as the **foreach** loop in example above but runs on multiple cores in parallel. E.g.
```csharp
    foreach (var (components, entities) in query.Chunks) { ... }
```
To enable running a query job a [ParallelJobRunner](https://github.com/friflo/Friflo.Engine-docs/blob/main/api/ParallelJobRunner.md) is required.  
The runner can be assigned to the `EntityStore` or directly to the `QueryJob`.  
A `ParallelJobRunner` instance is thread-safe and can / should be used for multiple / all query jobs.

```csharp
public static void ParallelQueryJob()
{
    var runner  = new ParallelJobRunner(Environment.ProcessorCount);
    var store   = new EntityStore { JobRunner = runner };
    for (int n = 0; n < 10_000; n++) {
        store.CreateEntity(new MyComponent());
    }
    var query = store.Query<MyComponent>();
    var queryJob = query.ForEach((myComponents, entities) =>
    {
        // multi threaded query execution running on all available cores
        for (int n = 0; n < entities.Length; n++) {
            myComponents[n].value += 10;
        }
    });
    queryJob.RunParallel();
    runner.Dispose();
}
```
In case of structural changes inside `ForEach((...) => {...})` use
[CommandBuffer.Synced](https://github.com/friflo/Friflo.Engine-docs/blob/main/api/CommandBuffer.Synced.md)
to record the changes.  
These changes are adding / removing components, tags or child entities and the creation / deletion of entities.

Note: `CommandBuffer` is **_not_** thread safe. `CommandBuffer.Synced` **_is_** thread safe.  
After `RunParallel()` returns these changes can be applied to the `EntityStore` by calling `CommandBuffer.Playback()`.

**Recommendation**  
A parallel query achieves notable performance gains in case using only arithmetic computations like * / + - sin(), cos(), ... in the loop.  

In case of using a `CommandBuffer` and and applying massive entity changes the single threaded version is typically faster.  
The reason is that entity changes applied to a `CommandBuffer` requires heavy random memory access.  
If doing this on multiple threads the CPU cores are competing with access to memory heap and CPU caches.

<br/>


## Query Vectorization - SIMD

**Optimization**: Utilize SIMD of your CPU to execute a `query`.  
SIMD architectures: SSE, AVX, AVX2, AVX-512, AdvSIMD, Neon, ...

The most efficient way to speedup query execution is vectorization.  
Vectorization is similar to loop unrolling - aka loop unwinding - but with hardware support.  
Its efficiency is superior to multi threading as it requires only a single thread to achieve the same performance gain.  
So other threads can still keep running without competing for CPU resources.  

*Note:* Vectorization can be combined with multi threading to speedup execution even more.  
In case of a system with high memory bandwidth the speedup is *speedup(SIMD) * speedup(multi threading)*.  
If SIMD or multi threading alone already reaches this bandwidth bottleneck their combination provide no performance gain.

The API provide a few methods to convert chunk components into [System.Runtime.Intrinsics - Vectors](https://learn.microsoft.com/en-us/dotnet/api/system.runtime.intrinsics).  
E.g. `AsSpan256<>` and `StepSpan256`. See all methods at the [Chunk - API](https://github.com/friflo/Friflo.Engine-docs/blob/main/api/Chunk_T_.md).  
The `Span` retrieved from a  chunk component has padding components at the end to enable vectorization without a scalar remainder loop.

The following examples shows how to increment all `MyComponent.value`'s by 1.  

```csharp
public static void QueryVectorization()
{
    var store   = new EntityStore();
    for (int n = 0; n < 10_000; n++) {
        store.CreateEntity(new MyComponent());
    }
    var query = store.Query<MyComponent>();
    foreach (var (component, entities) in query.Chunks)
    {
        // increment all MyComponent.value's. add = <1, 1, 1, 1, 1, 1, 1, 1>
        var add     = Vector256.Create<int>(1);     // create int[8] vector - all values = 1
        var values  = component.AsSpan256<int>();   // values.Length - multiple of 8
        var step    = component.StepSpan256;        // step = 8
        for (int n = 0; n < values.Length; n += step) {
            var slice   = values.Slice(n, step);
            var result  = Vector256.Create<int>(slice) + add; // 8 add ops in one CPU cycle
            result.CopyTo(slice);
        }
    }
}
```
<br/>


## EventFilter

**Optimization**: Process multiple entity events in a loop instead of individual event handlers.

An alternative to process entity changes - see section
[Event](../examples/General.md#event) - are `EventFilter`'s.  
`EventFilter`'s can be used on its own or within a query like in the example below.  
All events that need to be filtered - like added/removed components/tags - can be added to the `EventFilter`.  
E.g. `ComponentAdded<Position>()` or `TagAdded<MyTag1>`.  

```csharp
public static void FilterEntityEvents()
{
    var store   = new EntityStore();
    store.EventRecorder.Enabled = true; // required for EventFilter
    
    store.CreateEntity(new Position(), Tags.Get<MyTag1>());
    
    var query = store.Query();
    query.EventFilter.ComponentAdded<Position>();
    query.EventFilter.TagAdded<MyTag1>();
    
    foreach (var entity in store.Entities)
    {
        bool hasEvent = query.HasEvent(entity.Id);
        Console.WriteLine($"{entity} - hasEvent: {hasEvent}");
    }
    // > id: 1  [] - hasEvent: False
    // > id: 2  [Position] - hasEvent: True
    // > id: 3  [#MyTag1] - hasEvent: True
}
```
<br/>


# Batching

## Batch - Create Entity

🔥 **Update** - New example to use simpler / more performant approach to create entities.

**Optimization**  
Minimize structural changes when creating entities.

Entities can be created with multiple components and tags in a single step.  
This can be done by one of the EntityStoreExtensions
[CreateEntity<T1, ... , Tn>](https://github.com/friflo/Friflo.Engine-docs/blob/main/api/EntityStoreExtensions.md) overloads.

```csharp
public static void CreateEntityOperation()
{
    var store   = new EntityStore();
    for (int n = 0; n < 10; n++) {
        store.CreateEntity(new EntityName("test"), new Position(), Tags.Get<MyTag1>());
    }
    var taggedEntities = store.Query().AllTags(Tags.Get<MyTag1>());
    Console.WriteLine(taggedEntities);      // > Query: [#MyTag1]  Count: 10
}
```
<br/>


## Bulk - Create entities

🔥 **Update** - New example for bulk creation of entities.

**Optimization**  
Create multiple entities with the same set of components / tags in a single step.

Entities can be created one by one with `store.CreateEntity()`.  
To create multiple entities with the same set of components and tags use  
[archetype.CreateEntities(int count)](https://github.com/friflo/Friflo.Engine-docs/blob/main/api/Archetype.CreateEntities(int).md).

```csharp
public static void CreateEntities()
{
    var store     = new EntityStore();
    var archetype = store.GetArchetype(ComponentTypes.Get<Position, Scale3>(), Tags.Get<MyTag1>());
    var entities  = archetype.CreateEntities(100_000);  // ~ 0.5 ms
    Console.WriteLine(entities.Count);                  // 100000
}
```
<br/>


## Batch - Operations

🔥 **Update** - Add example to batch **add**, **remove** and **get** components and tags.

**Optimization**  
Minimize structural changes when adding / removing **multiple** components or tags.

Components can be added / removed **one by one** to / from an entity with  
`entity.AddComponent()` / `entity.RemoveComponent()`.  
Every operation may cause a structural change which is an expensive operation.

To execute these operations in a single step use the
[EntityExtensions overloads](https://github.com/friflo/Friflo.Engine-docs/blob/main/api/EntityExtensions.md).  
This approach also reduces the amount of code to perform these operations.

In case accessing multiple components of the same entity use [entity.Data](https://github.com/friflo/Friflo.Engine-docs/blob/main/api/EntityData.md)
instead of multiple `entity.GetComponent<>()` calls.

```csharp
public static void EntityBatchOperations()
{
    var store   = new EntityStore();
    var entity  = store.CreateEntity();
    
    // batch: add operations
    entity.Add(
        new Position(1, 2, 3),
        new Scale3(4, 5, 6),
        new EntityName("test"),
        Tags.Get<MyTag1>());
    Console.WriteLine(entity); // id: 1  "test"  [EntityName, Position, Scale3, #MyTag1]
    
    // batch: get operations
    var data    = entity.Data; // introduced in 3.0.0-preview.7
    var pos     = data.Get<Position>();
    var scale   = data.Get<Scale3>();
    var name    = data.Get<EntityName>();
    var tags    = data.Tags;
    Console.WriteLine($"({pos}),({scale}),({name})"); // (1, 2, 3), (4, 5, 6), ('test')
    Console.WriteLine(tags);                          // Tags: [#MyTag1]
    
    // batch: remove operations
    entity.Remove<Position, Scale3, EntityName>(Tags.Get<MyTag1>());
    Console.WriteLine(entity); // id: 1  []
}
```

## Batch - Entity

**Optimization**  
Minimize structural changes when adding **and** removing components or tags to / from a **single entity**.

**Note**  
An `EntityBatch` should only be used when adding **AND** removing components / tags to an entity at the same entity.  
If only adding **OR** removing components / tags use the **Add()** / **Remove()** overloads shown above.

When adding/removing components or tags to/from a single entity it will be moved to a new archetype.  
This is also called a *structural change* and in comparison to other methods a more costly operation.  
Every component / tag change will cause a *structural change*.

In case of multiple changes on a single entity use an [EntityBatch](https://github.com/friflo/Friflo.Engine-docs/blob/main/api/EntityBatch.md)
to apply all changes at once.  
Using this approach only a single or no *structural change* will be executed.

```csharp
public static void EntityBatch()
{
    var store   = new EntityStore();
    var entity  = store.CreateEntity();
    
    entity.Batch()
        .Add(new Position(1, 2, 3))
        .AddTag<MyTag1>()
        .Apply();
    
    Console.WriteLine($"entity: {entity}"); // > entity: id: 1  [Position, #MyTag1]
}
```
<br/>


## EntityBatch - Query

**Optimization**: Minimize structural changes when adding / removing components or tags to / from **multiple entities**.

In cases you need to add/remove components or tags to entities returned by a query use a **bulk operation**.  
Executing these type of changes are most efficient using a bulk operation.  
This can be done by either using `ApplyBatch()` or a common `foreach ()` loop as shown below.  
To prevent unnecessary allocations the application should cache and reuse the list instance for future batches.

```csharp
public static void BulkBatch()
{
    var store   = new EntityStore();
    for (int n = 0; n < 1000; n++) {
        store.CreateEntity();
    }
    var batch = new EntityBatch();
    batch.Add(new Position(1, 2, 3)).AddTag<MyTag1>();
    store.Entities.ApplyBatch(batch);
    
    var query = store.Query<Position>().AllTags(Tags.Get<MyTag1>());
    Console.WriteLine(query);               // > Query: [Position, #MyTag1]  Count: 1000
    
    // Same as: store.Entities.ApplyBatch(batch) above
    foreach (var entity in store.Entities) {
        batch.ApplyTo(entity);
    }
}
```
<br/>


## EntityBatch - EntityList

An [EntityList](https://github.com/friflo/Friflo.Engine-docs/blob/main/api/EntityList.md) is a container
of entities added to the list.  
Single entities are added using `Add()`. `AddTree()` adds an entity and all its children including their children etc.  
A **bulk operation** can be applied to all entities in the lists as shown in the example below.  


```csharp
public static void EntityList()
{
    var store   = new EntityStore();
    var root    = store.CreateEntity();
    for (int n = 0; n < 10; n++) {
        var child = store.CreateEntity();
        root.AddChild(child);
        // Add two children to each child
        child.AddChild(store.CreateEntity());
        child.AddChild(store.CreateEntity());
    }
    var list = new EntityList(store);
    // Add root and all its children to the list
    list.AddTree(root);
    Console.WriteLine($"list - {list}");        // > list - Count: 31
    
    var batch = new EntityBatch();
    batch.Add(new Position());
    list.ApplyBatch(batch);
    
    var query = store.Query<Position>();
    Console.WriteLine(query);                   // > Query: [Position]  Count: 31
}
```
<br/>


## CommandBuffer

**Optimization**: Required when adding / removing components or tags in a [Parallel Query Job](#parallel-query-job).

A `CommandBuffer` is used to record changes on multiple entities. E.g. `AddComponent()`.  
These changes are applied to entities when calling `Playback()`.    
Recording commands with a `CommandBuffer` instance can be done on **any** thread.  
`Playback()` must be called on the **main** thread.  
Available commands are in the [CommandBuffer - API](https://github.com/friflo/Friflo.Engine-docs/blob/main/api/CommandBuffer.md).  

This enables recording entity changes in multi threaded application using entity systems / queries.  
In this case enumerations of query results run on multiple worker threads.  
Within these enumerations entity changes are recorded with a `CommandBuffer`.  
After a query thread has finished these changes are executed with `Playback()` on the **main** thread.

```csharp
public static void CommandBuffer()
{
    var store   = new EntityStore();
    var entity1 = store.CreateEntity(new Position());
    var entity2 = store.CreateEntity();
    
    CommandBuffer cb = store.GetCommandBuffer();
    var newEntity = cb.CreateEntity();
    cb.DeleteEntity  (entity2.Id);
    cb.AddComponent  (newEntity, new EntityName("new entity"));
    cb.RemoveComponent<Position>(entity1.Id);        
    cb.AddComponent  (entity1.Id, new EntityName("changed entity"));
    cb.AddTag<MyTag1>(entity1.Id);
    
    cb.Playback();
    
    var entity3 = store.GetEntityById(newEntity);
    Console.WriteLine(entity1);     // > id: 1  "changed entity"  [EntityName, #MyTag1]
    Console.WriteLine(entity2);     // > id: 2  (detached)
    Console.WriteLine(entity3);     // > id: 3  "new entity"  [EntityName]
}
```