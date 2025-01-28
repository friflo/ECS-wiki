
The straight forward way to execute a query is using `query.ForEachEntity()`.  
This implementation is super compact and provide a direct result during development. E.g.

```csharp
// query creation
var query = store.Query<EntityName>();

// query execution
query.ForEachEntity((ref EntityName name, Entity entity) => {
    ....
});
```
This execution type creates a delegate (lambda) and the callback for every result add additional execution costs.

There are various ways to improve query execution significant.  
Query creation remains unchanged for all variants by using the same `Query<>()` instance.  
```csharp
    var query = store.Query<EntityName>();
```
Only the code related for query execution is different and require additional boilerplate code.  
This approach enables:
- Query optimization can be postponed. E.g. a specific query turns out to be a bottleneck.
- All optimization variants require only local code changes.

Query optimizations can be achieved by using various query technics like:
- [Boosted Query](#boosted-query)
- [Enumerate Query Chunks](#enumerate-query-chunks)
- [Parallel Query Job](#parallel-query-job)
- [Query Vectorization - SIMD](#query-vectorization---simd)


Query optimization examples are part of the unit tests see:
[Tests/ECS/Examples](https://github.com/friflo/Friflo.Engine.ECS/tree/main/src/Tests/ECS/Examples)

> **Info**  
> In general *friflo ECS* provides in many areas only a single API to implement a specific behavior.  
> This avoids confusion to select a one of multiple API variants if available.  
> **Query optimization** and **Batch operations** are an exception to this rule.  
> The reason is to provide provide room for significant performance gains if needed.

<br/>

## Boosted Query


🔥 **Update** - Introduced new query approach using [Friflo.Engine.ECS.Boost](https://www.nuget.org/packages/Friflo.Engine.ECS.Boost#readme-body-tab).

**Optimization**: Use `query.Each()` and a struct implementing `Execute(...)` for maximum query performance.  

This query approach is the most performant approach - except Query vectorization / SIMD.  
A unique feature of **Friflo.Engine.ECS** - it uses no **unsafe code**. This enables running the dll in trusted environments.  
For maximum performance unsafe code is required to elide bounds checks.  

Instead of processing components of a query with `query.ForEachEntity(...)`  
the `MoveEach` struct below process components in its `Execute()` method.  
The processing of all query components is performed by `query.Each(new MoveEach())`.

The performance gain compared with `query.ForEachEntity(...)` is ~3x.  
The method `query.Each()` requires adding the dependency [Friflo.Engine.ECS.Boost](https://www.nuget.org/packages/Friflo.Engine.ECS.Boost#readme-body-tab).

```cs
public struct Velocity : IComponent { public Vector3 value; }

public static void BoostedQuery()
{
    var store   = new EntityStore();
    for (int n = 0; n < 100; n++) {
        store.CreateEntity(new Position(), new Velocity());
    }
    var query = store.Query<Position, Velocity>();
    query.Each(new MoveEach()); // requires https://www.nuget.org/packages/Friflo.Engine.ECS.Boost
}

struct MoveEach : IEach<Position, Velocity>
{
    public void Execute(ref Position position, ref Velocity velocity) {
        position.value += velocity.value;
    }
} 
```

<br/>


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
🔥 **Update** - Added example code for slower iteration alternative.  
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

**Re-use QueryJob**

The `QueryJob` returned by `query.ForEach(...)` is intended to be re-used.  
This ensures that subsequent calls to `queryJob.RunParallel()` do not perform allocations on GC heap.

**Recommendation**  
A parallel query achieves notable performance gains in case using only arithmetic computations like * / + - sin(), cos(), ... in the loop.  

In case of using a `CommandBuffer` and and applying massive entity changes the single threaded version is typically faster.  
The reason is that entity changes applied to a `CommandBuffer` requires heavy random memory access.  
If doing this on multiple threads the CPU cores are competing with access to memory heap and CPU caches.

<br/>


## Query Vectorization / SIMD

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