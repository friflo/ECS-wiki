<div style="display: flex;">
<div style="flex-grow: 1;"></div>
<div>
<a href="https://github.com/friflo/Friflo.Engine.ECS"/><img src="../images/friflo-ECS-small.svg"></a>
<span width="40" style="display:inline;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span>
<a href="https://github.com/friflo/Friflo.Engine.ECS"/><img src="../images/github-mark.svg"></a>
<a href="https://github.com/friflo/Friflo.Engine.ECS"/><img src="https://img.shields.io/badge/GitHub-grey"></a>
<span width="40" style="display:inline;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span>
<a href="https://discord.gg/nFfrhgQkb8"/><img src="../images/discord.svg"></a>
<a href="https://discord.gg/nFfrhgQkb8"/><img src="https://img.shields.io/badge/Discord-5865F2"></a>
</div>
</div>


Features in this list are also explained in the GitHub Wiki.  
- [**Examples - General**](../examples/General.md)  
- [**Examples - Optimization**](../examples/Optimization.md)  

Every new version is backward compatible to earlier versions.  
Exceptions are labeled as  **Breaking change** / **Changed behavior**. These changes are made only on rarely used features.

Releases on nuget are available at  
[![nuget](https://img.shields.io/nuget/v/Friflo.Engine.ECS?logoColor=white)](https://www.nuget.org/packages/Friflo.Engine.ECS)

Detailed information for each release at
[GitHub - Release Tags](https://github.com/friflo/Friflo.Engine.ECS/releases)

<br/>

# 3.x Releases

## 3.3.0

The main focus of this release was to add support for **EcGui** - an In-Game GUI utilizing ImGui.NET.  
The GUI is an optional extension and acts as an overlay over the Game screen.  
It provides instant access to entities their components, tags and relations at runtime via **Explorer** and **Inspector** window.

### New Features
To enable sorting and filtering in **EcGui** the following features were added to the ECS.
- Support sorting an `EntityList` with `SortByComponentField<,>()` and `SortByEntityId()`.
- Support to filter an `EntityList` using `Filter(Func<Entity, bool> filter)` with a predicate delegate.

### Bugfix
- Fixed [Native AOT - System.NotSupportedException](https://github.com/friflo/Friflo.Engine.ECS/issues/65)


### **EcGui**

**EcGui** can be integrated in all environments that support **ImGui**. This means all desktop platforms: Windows, macOS & Linux.  
It supports various graphic backends: Direct3D, Vulkan, Metal, OpenGL, ...

#### Features
- Requires only a few method calls (3 at minimum) to connect  **ECS** store, queries and systems to **EcGui**.
  ```cs
  // on startup
  EcGui.AddExplorerStore("Store", store);
  // in render loop
  EcGui.ExplorerWindow();
  EcGui.InspectorWindow();
  ```
- Renders the Explorer & Inspector in ~0.1 - 0.5 ms per frame. Rendering is allocation free.  
  The time for rendering is independent from entity count. E.g. 1, 10, ..., 1.000 or 1.000.000 entities.
- Explore ECS queries added to the **Explorer** in real time.
- Filter and sort entity components in the explorer table.
- Edit, Cut, Copy or Paste entities and components in the **Explorer** table.
- Export Explorer table as **CSV** or **Markdown**.
- View and edit all entity components, tags and relations in the **Inspector** window.


Minimal Demos showing how to integrate
**EcGui** in
[MonoGame](https://github.com/friflo/friflo-EcGui-MonoGame),
[Godot](https://github.com/friflo/friflo-EcGui-Godot),
[SDL3](https://github.com/friflo/friflo-EcGui-SDL3.GPU) &
[Silk.NET](https://github.com/friflo/friflo-EcGui-Silk.NET.OpenGL)  
See: [**EcGui Demo List**](https://github.com/friflo?tab=repositories&q=ecgui&type=public&language=&sort=name)

![friflo-EcGui-MonoGame](https://github.com/user-attachments/assets/5150eafe-fad8-4502-88c9-7ceb9b60cbc6)


## 3.2.1
Simplified API
- Added `Entity.CopyEntity(Entity target)`  
  Intended to backup the entities from one `EntityStore` to another.  
  See: [Docs - Clone / Copy entity](https://friflo.gitbook.io/friflo.engine.ecs/documentation/entity#clone-copy-entity)
- Added `Entity.CloneEntity()`

## 3.2.0
- New feature: added `EntityStore.CopyEntity(Entity source, Entity target)`

## 3.1.0
- Introduced `StructuralChangeException` which is thrown when executing **structural changes** within a `Query<>()` loop.  
  Detailed information at [Query - StructuralChangeException](https://friflo.gitbook.io/friflo.engine.ecs/documentation/query#structuralchangeexception)
- Component type `UniqueEntity` now implements `IIndexedComponent<string>` to improve performance of `EntityStore.GetUniqueEntity(string uid)`.

## 3.0.4
- `RelationsEnumerator<>.Current` returns entities now by ref. This enables to modify relations in a `GetRelations<>()` loop.
   E.g.
  ```cs
    foreach (ref var relation in entity.GetRelations<MyRelation>()) {
        relation.Value += 1;
    }
  ```

## 3.0.3
- Add Native AOT support for specialized component types and relations introduced in v3.0.0:  
  `IIndexedComponent<>`, `ILinkComponent`, `IRelation<>` and `ILinkRelation`.  
  These types must be registered with: `NativeAOT.RegisterIndexedComponent()` or `NativeAOT.RegisterRelation()`.


## 3.0.2
- Implemented `Entity.Equals(object)` and `Entity.GetHashCode()` to enable use in `Dictionary<Entity,>` and `HashSet<Entity>`.  
  See [FitHub Issue#56 - Entity.Equals() throws "Not implemented to avoid excessive boxing."](https://github.com/friflo/Friflo.Engine.ECS/issues/56)
- Fixed [GitHub Issue#55 - SystemRoot.GetPerfLog() throws ArgumentOutOfRange when system names are too long.](https://github.com/friflo/Friflo.Engine.ECS/issues/55).  
  The length of the System name column can now be customized in `system.GetPerfLog(int nameColLen)`.

## 3.0.1

- Simplify - Made `BaseSystem.OnAddStore()` and `BaseSystem.OnRemoveStore()` noop methods.  
  An `override` of these method are not required to call the base implementation anymore which happened accidentally.
- Fixed `AsSpan...<>()` when used in multithreaded queries. See https://github.com/friflo/Friflo.Engine.ECS/issues/47

## 3.0.0

Finally after 6 months in **preview** all new features are completed.
- Updated test dependencies and ensured compatibility with MonoGame, Godot, Unity (Mono, IL2CPP & WebGL) and Native AOT.

## 3.0.0-preview.19

- Changed implementation of `EntityStore.CloneEntity()`  

  **Old:** Used JSON serialization as fallback if source entity has one or more *non-blittable* component types.  
         E.g. a struct component containing fields with reference types like a `array`, `List<>` or `Dictionary<,>`.  

  **New:** *Non-blittable* components are now cloned using a static `CopyValue()` method with must part of component type.
  E.g.
  ```cs
  public struct MyComponent : IComponent
  {
      public int[]   array;
    
      static void CopyValue(in MyComponent source, ref MyComponent target, in CopyContext context) {
          target.array = source.array?.ToArray();
      }
  }
  ```
  In case `CopyValue()` is missing an exception is thrown including the required method signature in the exception message.  
  *Info:* Removing JSON serialization from cloning has additional advantages.
  1. Much faster as JSON serialization/deserialization is expensive.
  2. No precision loss when using floating point types.
  

## 3.0.0-preview.18

- Changed license to MIT. Used LGPL before.
- Create `ComponentIndex` to simplify search entities with specific components in O(1).
  ```cs
    var store = new EntityStore();
    store.CreateEntity(new Player { name = "Player One" });
    var index = store.ComponentIndex<Player,string>();
    var entities = index["Player One"];         // Count: 1
  ```
- Create `EntityRelations` to simplify iteration of relations.


## 3.0.0-preview.17

- Added support of JSON serialization for [relations](../examples/Component-Types.md#serialization).
- Added row `M` to systems perf log to indicate if `MonitorPerf` is enabled for a specific system.


## 3.0.0-preview.16

> Breaking changes

1. To avoid mixing up relations with components accidentally `IRelation` does not extends `IComponent` anymore.
2. Renamed public API's  
   `IRelationComponent<>` -> `IRelation<>`  
   `RelationComponents<>` -> `Relations<>`


## 3.0.0-preview.15

Added support of serialization of `Entity` fields in components. E.g.
```cs
struct RefComponent : IComponent {
    public Entity reference;
}
```
will be serialized as JSON
```json
{
    "components": {
        "RefComponent": {"reference":42}
    }  
}
```


## 3.0.0-preview.14

Fixes issues specific to indexed components and relations.
- [Indexed component not found when adding component through EntityStore.CreateEntity - Issue #5](https://github.com/friflo/Friflo.Engine.ECS/issues/5)
- [After deleting the entity, the target entity still has incoming links - Issue #13](https://github.com/friflo/Friflo.Engine.ECS/issues/13)
- [After serialization/deserialization indexed components don't work - Issue #15](https://github.com/friflo/Friflo.Engine.ECS/issues/15)
- [EntitySerializer throws when using a MemoryStream created from a byte[] - Issue #27](https://github.com/friflo/Friflo.Engine.ECS/issues/27)
- [New entities have the same parent of recycled entities - Issue #29](https://github.com/friflo/Friflo.Engine.ECS/issues/29)
- [Use CommandBuffer.SetComponent then crash in Playback- Issue #31](https://github.com/friflo/Friflo.Engine.ECS/issues/31)


## 3.0.0-preview.11

Return the passed `each` parameter in `QueryExtensions.Each(each)` methods.  
This enables access to the state of the structs implementing `IEach<...>`.
```cs
    var query = store.Query<Position, Velocity>();
    var each  = query.Each(new CountEach()); // returns the CountEach parameter
    WriteLine(each.count);                   // in case CountEach count invocations
```

## 3.0.0-preview.10

*Same as 3.0.0-preview.8, 3.0.0-preview.9. Created until CI was successful*

Published new nuget package [Friflo.Engine.ECS.Boost](https://www.nuget.org/packages/Friflo.Engine.ECS.Boost/3.0.0-preview.10#readme-body-tab)  
A unique feature of **Friflo.Engine.ECS** is - it uses no **unsafe code**.  
For maximum query performance unsafe code it required to elide array bounds checks.  
The new Boost package contains an extension dll and make use of unsafe code to improve query performance for large results ~30%.  
See [Boosted Query](../examples/Optimization.md#boosted-query) with example used for maximum query performance.

## 3.0.0-preview.7
Focus of preview.7 was performance improvements

- **Changed behavior** The ids of deleted entities are now recycled when creating new entities.  
  Before: Every created entity got its own (incrementing ) unique id. This behavior lead to an every growing buffer when creating new entities.  
  To switch to old behavior set [EntityStore.RecycleIds](https://github.com/friflo/Friflo.Engine-docs/blob/main/api/EntityStore.RecycleIds.md) = false.

- Introduced [EntityData](https://github.com/friflo/Friflo.Engine-docs/blob/main/api/EntityData.md) struct to optimize
  read / write access to multiple components of the same entity.  
  The cost to access a component is significant less than `Entity.GetComponent()`.

- Change policy used to shrink the capacity of archetype containers.  
  See: [EntityStore.ShrinkRatioThreshold](https://github.com/friflo/Friflo.Engine-docs/blob/main/api/EntityStoreBase.ShrinkRatioThreshold.md)

## 3.0.0-preview.6
- Same release as preview.5 build with GitHub Action in [new Repository](https://github.com/friflo/Friflo.Engine.ECS)

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
  See documentation at: [Examples - Component-Types](../examples/Component-Types.md)

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
![new](../images/new.svg) **Features**
- Introduced Systems, System Groups with command buffers and performance monitoring.  
Details at [README - Systems](https://github.com/friflo/Friflo.Engine.ECS/blob/main/README.md#%EF%B8%8F-systems)
- Added support for Native AOT.  
Details at [Wiki ⋅ General - Native AOT](../examples/General.md#native-aot).
- Enabled sharing a single [QueryFilter](https://github.com/friflo/Friflo.Engine-docs/blob/main/api/QueryFilter.md) instance by multiple queries.  
Changing a query filter - e.g. adding more constrains - directly changes the result set of all queries using the `queryFilter`.
```cs
var query = store.Query<....>(queryFilter);
```
- Added [CommandBuffer.Synced](https://github.com/friflo/Friflo.Engine-docs/blob/main/api/CommandBuffer.Synced.md) intended to record entity changes in [parallel query jobs](../examples/General.md#parallel-query-job).

**Performance**
- Improved bulk creation of entities by 3x - 4x with [Archetype.CreateEntities(int count)](https://github.com/friflo/Friflo.Engine-docs/blob/main/api/Archetype.CreateEntities(int).md).  
See performance comparison at [ECS Benchamrks](https://github.com/friflo/Friflo.Engine.ECS?tab=readme-ov-file#-ecs-benchmarks).
- Reduced memory footprint of an entity from 48 bytes to 16 bytes.  
See column **Allocated** in [ECS Benchamrks](https://github.com/friflo/Friflo.Engine.ECS?tab=readme-ov-file#-ecs-benchmarks).
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
Reduced number of files / folders in `Engine folder` from 21 to 7.
- Moved documentation and examples from `README.md` to [GitBook.io ⋅ Wiki](https://friflo.gitbook.io/friflo.engine.ecs) pages.

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

