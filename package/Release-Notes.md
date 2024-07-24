

Features in this list are also explained in the GitHub Wiki.  
- [**Examples - General**](https://friflo.gitbook.io/ecs-wiki/Examples-General)  
- [**Examples - Optimization**](https://friflo.gitbook.io/ecs-wiki/Examples-Optimization)  

Every new version is backward compatible to earlier versions.  
Exceptions are labeled as  **Breaking change** / **Changed behavior**. These changes are made only on rarely used features.

Releases on nuget are available at  
[![nuget](https://img.shields.io/nuget/v/Friflo.Engine.ECS?logo=nuget&logoColor=white)](https://www.nuget.org/packages/Friflo.Engine.ECS)

Detailed information for each release at
[GitHub - Release Tags](https://github.com/friflo/Friflo.Json.Fliox/releases)

<br/>

# 3.x Releases

## 3.0.0-preview.5
- Improved performance of `Entity.RemoveChild()` & `Entity.InsertChild()` when remove / insert child entity on tail.

## 3.0.0-preview.4
- Added runtime assertions to ensure an entity tree (parent / child relations) never contains cycles.  
  So the tree is always a [DAG - directed acyclic graph](https://en.wikipedia.org/wiki/Tree_(graph_theory)).  
  An operation e.g. `AddChild()` which would create a cycle within a tree will throw an exception like:
  ```
  System.InvalidOperationException : operation would cause a cycle: 3 -> 2 -> 1 -> 3
  ```
- Improved performance of `Entity.AddChild()`, `Entity.RemoveChild()` & `Entity.InsertChild()` by 30%

## 3.0.0-preview.3
- Reduced number of properties displayed for an Entity in the debugger.  
  Moved properties: Archetype, Parent & Scripts  to Entity.Info

- Improved performance and memory allocations to build a scene tree.  
  Each parent in a tree has a `TreeNode` component.  
  reduced `sizeof(TreeNode)` component from 16 to 8 bytes.

  *Before:* Each `TreeNode` has an individual int[] array storing child ids.  
  *Now:* child ids are stored in a ~ dozen id pool array buffers. Independent from the number of parent or child entities.  
  If these array buffers grown large enough over time no heap allocations will happen if adding or removing child entities.

## 3.0.0-preview.2
- For specific use cases there is now a set of specialized component interfaces providing additional features.    
  *Note:* Newly added features do not affect the behavior or performance of existing features.  
  See documentation at: [Examples - Component-Types](https://friflo.gitbook.io/ecs-wiki/Examples-Component-Types)

  The specialized component types enable entity relationships, relations and full-text search.  
  Typical use case for entity relationships in games are:
  - Attack systems
  - Path finding / Route tracing
  - Model social networks. E.g friendship, alliances or rivalries
  - Build any type of a [directed graph](https://en.wikipedia.org/wiki/Directed_graph)
  using entities as *nodes* and links or relations as *edges*.


<br/>

# 2.x Releases

## 2.1.0
Added support for generic component and tags types.
See [Issue #53](https://github.com/friflo/Friflo.Json.Fliox/issues/53). E.g.
```cd
public struct GenericComponent<T> : IComponent {
    public T Value;
}
```

## 2.0.0
![new](images/new.svg) **Features**
- Introduced Systems, System Groups with command buffers and performance monitoring.  
Details at [README - Systems](https://github.com/friflo/Friflo.Json.Fliox/blob/main/Engine/README.md#systems)
- Added support for Native AOT.  
Details at [Wiki ⋅ General - Native AOT](https://friflo.gitbook.io/ecs-wiki/Examples-General#native-aot).
- Enabled sharing a single [QueryFilter](https://github.com/friflo/Friflo.Engine-docs/blob/main/api/QueryFilter.md) instance by multiple queries.  
Changing a query filter - e.g. adding more constrains - directly changes the result set of all queries using the `queryFilter`.
```cs
var query = store.Query<....>(queryFilter);
```
- Added [CommandBuffer.Synced](https://github.com/friflo/Friflo.Engine-docs/blob/main/api/CommandBuffer.Synced.md) intended to record entity changes in [parallel query jobs](https://friflo.gitbook.io/ecs-wiki/Examples-Optimization#parallel-query-job).

**Performance**
- Improved bulk creation of entities by 3x - 4x with [Archetype.CreateEntities(int count)](https://github.com/friflo/Friflo.Engine-docs/blob/main/api/Archetype.CreateEntities(int).md).  
See performance comparison at [ECS Benchamrks](https://github.com/friflo/Friflo.Json.Fliox/blob/main/Engine/README.md#ecs-benchmarks).
- Reduced memory footprint of an entity from 48 bytes to 16 bytes.  
See column **Allocated** in [ECS Benchamrks](https://github.com/friflo/Friflo.Json.Fliox/blob/main/Engine/README.md#ecs-benchmarks).
- Decreased initial component type registration from 80 ms to 23 ms (Mac Mini M2).  
Note: Component type registration runs only once per process on the first ECS API call.

**Fixes**
- Fixed issue related to JSON serialization. See [Issue #45](https://github.com/friflo/Friflo.Json.Fliox/issues/45)  
**Before**: Component types with unsupported field types caused an exception.  
**Fix**: These unsupported field types are now ignored by JSON serializer.
- Parallel queries: fixed entities - of type `ChunkEntities` - used as last parameter in a `ArchetypeQuery.ForEach()`.  
  See  [Issue #50](https://github.com/friflo/Friflo.Json.Fliox/issues/50)
```cs
query.ForEach((..., entities) => { ... });
```
Before: `entities` returned always the same set of entities.
Fix: `entities` now returns the entities assigned to a thread.
- Adding a Signal handler to an entity already having a Signal handler replaced the existing one.  
  See  [Issue #46](https://github.com/friflo/Friflo.Json.Fliox/issues/46)

**Project**
- Simplify project structure for **Friflo.Engine.ECS**.
Reduced number of files / folders in [Engine](https://github.com/friflo/Friflo.Json.Fliox/tree/main/Engine) folder from 21 to 7.
- Moved documentation and examples from `README.md` to [GitHub ⋅ Wiki](https://friflo.gitbook.io/ecs-wiki) pages.

<br/>

# 1.x Releases

## 1.28.0
Optimize performance of `Add<>()`, `Set<>()` and `Remove<>()` introduced in **1.26.0**.

## 1.26.0
Features:  
Add 10 `CreateEntity()` overloads to create entities with components without any structural change in [EntityStoreExtensions](https://github.com/friflo/Friflo.Engine-docs/blob/main/api/EntityStoreExtensions.md).  
Add 10 overloads to `Add<>()`, `Set<>()` and `Remove<>()` entity components with one/none structural change in [EntityExtensions](https://github.com/friflo/Friflo.Engine-docs/blob/main/api/EntityExtensions.md).  
Add [ArchetypeQuery.ToEntityList()](https://github.com/friflo/Friflo.Engine-docs/blob/main/api/ArchetypeQuery.ToEntityList().md) returning the entities as a list which can be used for structural changes.  
Emit events on create / delete entity via [EntityStore.OnEntityCreate](https://github.com/friflo/Friflo.Engine-docs/blob/main/api/EntityStore.OnEntityCreate.md) and
[EntityStore.OnEntityDelete](https://github.com/friflo/Friflo.Engine-docs/blob/main/api/EntityStore.OnEntityDelete.md).

## 1.25.0
Switched project to more permissive license LGPL v3.0. Before AGPL v3.0.
See [Issue #41](https://github.com/friflo/Friflo.Json.Fliox/discussions/41).

## 1.24.0
Add support for component fields of type: `sbyte, ushort, uint, ulong`.
See [Issue #38](https://github.com/friflo/Friflo.Json.Fliox/issues/38).

## 1.23.0
Support integration in Unity as nuget package.  
Supports Mono & AOT/IL2CPP builds. Tested on Windows & macOS.

## 1.19.0
Add `ArchetypeQuery.ForEachEntity()` for convenient query iteration.  
Support / fix using vector types - e.g. `Vector3` - as component fields for .NET 7 or higher.  

## 1.18.0
Introduced `EntityList` to apply an entity batch to all entities in the list.  
Add `Entity.Enabled` to enable/disable an entity.  
Add `Entity.EnableTree()` / `Entity.DisableTree()` to enable/disable recursively the child entities of an entity.

## 1.17.0
Introduced `CreateEntityBatch` to optimize creation of entities.  
Added DebugView's for all IEnumerable<> types to enable one click navigation to their elements in the debugger.  
E.g. the expanded properties ChildEntities and Components in the examples screenshot.  
**Breaking change**: Changed property `Entity.Batch` to method `Entity.Batch()`.

## 1.16.0
Add support for entity batches and bulk batch operations to apply multiple entity changes at once.  
**Changed behavior** of the Archetype assigned to entities without components & tags.  
*Before:* Entities were not stored in this specific Archetype. `Archetype.Entities` returned always an empty result.  
*Now:*    Entities are stored in this Archetype.  

## 1.15.0
Reduced the number of properties shown for an entity in the debugger. See screenshot in Examples. 

## 1.14.0
Add support for parallel (multi threaded) query job execution.


## 1.13.0
Add support for target framework .NET Standard 2.1 or higher.


## 1.12.0
Add additional query filters like `WithoutAnyTags()` using an
[ArchetypeQuery](https://github.com/friflo/Friflo.Engine-docs/blob/main/api/ArchetypeQuery.md).  

## 1.11.0
Support to filter entity changes - like adding/removing components/tags - in queries using an
[EventFilter](https://github.com/friflo/Friflo.Engine-docs/blob/main/api/EventFilter.md).  


## 1.10.0
Add support for [CommandBuffer](https://github.com/friflo/Friflo.Engine-docs/blob/main/api/CommandBuffer.md)'s.  

