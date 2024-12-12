
Queries are a fundamental feature of an ECS.  

The major strength of an ECS is efficient / performant query execution.  
An Archetype based ECS store components in arrays. A query result provide direct access to these arrays - aka 'Chunks'.  
So the performance characteristic iterating a query result is the same as iterating an array.

> *Info*  
> Iterating arrays is the most efficient way to iterate large data sets by efficient use of the CPU L1 cache, its prefetcher and instruction pipelining.
> - 100% of the data to store components in **L1 cache lines** (typically 64 or 128 bytes) is utilized.
> - The **prefetcher** minimize caches misses as it detects the sequential array access which stores data in continuous memory.
> - Efficient use of **instruction pipelining** as array iteration require minimal conditional branches.

A query is build by specifying two aspects.

- The **components** a query returns when executed. They are passed a generic arguments to a `store.Query<>()`.  
  The query result contains only entities having all specified components.  
  *Info:* The specified components corresponds to the rows listed in a SQL `SELECT` statement.

- Add optionals [**query filters**](#query-filter) to reduce the query result only to entities matching the filters.  
  *Info:* The filters corresponds to SQL `WHERE` clause used to filter records.

The `ArchetypeQuery` returned by `store.Query<>()` is designed for reuse.  
It can be stored and reused to avoid the setup and allocation for a `Query<>()`.

```csharp
public static void EntityQueries()
{
    var store   = new EntityStore();
    store.CreateEntity(new EntityName("entity-1"));
    store.CreateEntity(new EntityName("entity-2"), Tags.Get<MyTag1>());
    store.CreateEntity(new EntityName("entity-3"), Tags.Get<MyTag1, MyTag2>());
    
    // --- query components
    var queryNames = store.Query<EntityName>();
    queryNames.ForEachEntity((ref EntityName name, Entity entity) => {
        // ... 3 matches
    });    
    // --- query components with tags
    var namesWithTags  = store.Query<EntityName>().AllTags(Tags.Get<MyTag1, MyTag2>());
    namesWithTags.ForEachEntity((ref EntityName name, Entity entity) => {
        // ... 1 match
    });    
    // --- use query.Entities in case an iteration requires no component access
    foreach (var entity in queryNames.Entities) {
        // ... 3 matches
    }
}
```
> **Note**  
> As mentioned above by storing components in arrays aka `Chunks` additional [Query Optimizations](query-optimization.md) can by applied if needed.


## Query Filter

## CommandBuffer

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