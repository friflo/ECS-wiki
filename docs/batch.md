
Batch operations can be used to improved performance and minimize *archetype fragmentation* when creating or modifying entities.  
For example the creation of an entity can be executed in multiple steps.
Each step moves the entity to a different archetype.

```cs
var entity = store. CreateEntity();
entity.AddComponent(new Position(1,2,3));
entity.AddComponent(new Transform());
entity.AddComponent(new EntityName("test"));
```

A Batch operation like `CreateEntity<>()` combine these steps in a single call and put the entity directly in the final archetype.
```cs
var entity = store.CreateEntity(new Position(1,2,3), new Transform(), new EntityName("test"));
```

*Terminology*
- A **Batch** combines multiple component changes in a single operation on the same entity.
- A **Bulk** operation performance the same operation on multiple entities.


# Create entities

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


# Add/Remove components

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

## Add/Remove components with `EntityBatch`

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


## `EntityBatch` - Bulk execution

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


## Apply `EntityBatch` to an `EntityList`

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
