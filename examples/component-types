
*Note:* This Wiki page is work in progress...

The common component type applicable for most use cases is
[IComponent](https://github.com/friflo/Friflo.Engine-docs/blob/main/api/IComponent.md).  
See component example in [Wiki ⋅ Examples General](../examples/General.md#component)

For specific use cases there is a set of component interfaces providing additional features.  

| Use case                                      | Component interface type                  | Description
| --------------------------------------------- | ----------------------------------------- | --------------------------------------------
| [Entity Relationships](#entity-relationships) | [Link Component](#link-component)         | A single link on an entity referencing another entity.
|                                               | [Link Relation](#link-relation)           | Multiple links on an entity referencing other entities.
| [Relations](#relations)                       | [Relation](#relation)                     | Add multiple components of the same type to an entity.
| [Search & Range queries](#search)             | [Indexed Component](#indexed-component)   | Full text search of component fields executed in O(1).<br/>Range queries on component fields having a sort order.


Component types examples using **Friflo.Engine.ECS** are part of the unit tests see:
[Tests/ECS/Examples](https://github.com/friflo/Friflo.Engine.ECS/tree/main/src/Tests/ECS/Examples)

<br/>


# Entity Relationships

An entity relationship is a *directed* link between two entities.

Typical use case for entity relationships in a game are:
- Attack systems
- Path finding / Route tracing
- Model social networks. E.g friendship, alliances or rivalries
- Inventory Systems
- Build any type of a [directed graph](https://en.wikipedia.org/wiki/Directed_graph)
  using entities as *nodes* and links or relations as *edges*.

In this ECS relationships are modeled as components.  
*Directed link* means that a link points from a **source** entity to a **target** entity.  
The entity containing a link component / relation is the **source** entity.

There are two interfaces used to define components with entity links:

1. [ILinkComponent](https://github.com/friflo/Friflo.Engine-docs/blob/main/api/ILinkComponent.md) -
   An entity can have only one link component at a time.

2. [ILinkRelation](https://github.com/friflo/Friflo.Engine-docs/blob/main/api/ILinkRelation.md) -
   An entity can have multiple link relations - one per **target** entity.

Now you might ask why having specialized component types for entity links.  
You can simply add an `Entity` field to a component type and you are done.  
This is absolutely correct but the specialized types provide the following features.

**Features of entity links**

- Get link component of an entity with `entity.GetComponent<AttackComponent>()` in O(1).

- Get link relations of an entity with `entity.GetRelations<AttackRelation>()` in O(1).

- Get entities including outgoing links referencing a specific **target** entity with  
  `target.GetIncomingLinks<AttackComponent>()` in O(1).  This make links **bidirectional**.

- Automatically removing links from all entities having a link to a **target** entity that is deleted.

- Get all entities in an EntityStore linked by a specific link component using  
  `store.LinkComponentIndex<AttackComponent>().Values` in O(1).

- Add multiple links to a single entity using [Link Relations](#link-relation).

- Show and navigate all incoming entity links in a debugger

<br/>

<img src="../images/entity-debugger-incoming-links.png" width="600" height="290"></img>  
*Screenshot:* Show and navigate all incoming entity links in a debugger at `Info.IncomingLinks`


**Comparison to implementations in other ECS projects.**  

Entity relationships in **flecs** and **BEVY** are modeled as component/entity pairs added to entities.  
The main differences are:

- In **flecs** links between entities can be created adhoc.  
  This ECS requires to define a specific component type to create links between entities.  
  This simplifies code navigation and establish a clear overview what types of links are used in a project.

- The API to create and query relations in **flecs** is very compact but not very intuitive - imho.  
  It is completely different from common component handling.
  See [flecs ⋅ Relationships](https://github.com/SanderMertens/flecs/blob/master/docs/Relationships.md)

- Adding, removing or updating a link does not cause [archetype fragmentation](https://www.flecs.dev/flecs/md_docs_2Relationships.html#fragmentation).  
  In **flecs** every relationship between two entities creates an individual archetype only containing a single entity / component.  
  So each entity relationship allocates ~ 1000 bytes required by the archetype stored in the heap for each link.  
  The more significant performance penalty is the side effect in queries. Many archetypes need to be iterated if they are query matches.

- Changing an entity link does not cause a structural change.  
  In **flecs** a new archetype needs to be created and the old archetype gets empty.  

Big shout out to [**fenn**ecs](https://github.com/outfox/fennecs) and [**flecs**](https://github.com/SanderMertens/flecs)
for the challenge to improve the feature set and performance of this project!

<br/>


## Link Component

A link component enables adding a link to another **target** entity.  
An entity can have only one link component at a time.  
Link components are added, removed and queried like common components with
```cs
    entity.AddComponent(new AttackComponent { target = entity2 });
    entity.GetComponent    <AttackComponent>();
    entity.RemoveComponent <AttackComponent>();
    entity.HasComponent    <AttackComponent>();
```

The following example illustrate the state changes by small text graphs on the right.  
Link components are represented by `→` icon.  
This example show the graph changes when adding link components or deleting entities.

```cs
struct AttackComponent : ILinkComponent {
    public  Entity  target;
    public  Entity  GetIndexedValue() => target;
}

public static void LinkComponents()
{
    var store   = new EntityStore();

    var entity1 = store.CreateEntity(1);                            // link components
    var entity2 = store.CreateEntity(2);                            // symbolized as →
    var entity3 = store.CreateEntity(3);                            //   1     2     3
    
    // add a link component to entity (2) referencing entity (1)
    entity2.AddComponent(new AttackComponent { target = entity1 }); //   1  ←  2     3
    // get all incoming links of given type.    O(1)
    entity1.GetIncomingLinks<AttackComponent>();        // { 2 }

    // update link component of entity (2). It links now entity (3)
    entity2.AddComponent(new AttackComponent { target = entity3 }); //   1     2  →  3
    entity1.GetIncomingLinks<AttackComponent>();        // { }
    entity3.GetIncomingLinks<AttackComponent>();        // { 2 }

    // deleting a linked entity (3) removes all link components referencing it
    entity3.DeleteEntity();                                         //   1     2
    entity2.HasComponent    <AttackComponent>();        // false
}
```
<br/>



## Link Relation

A link relation enables adding multiple links to a single entity referencing other **target** entities.  
There can be only one link relation per **target** entity.  
Link relations are added, removed and queried with
```cs
    entity.AddRelation(new AttackRelation { target = entity1 });
    entity.AddRelation(new AttackRelation { target = entity2 });
    entity.RemoveRelation <AttackRelation>(entity1);
    entity.GetRelations   <AttackRelation>();               // O(1)
    entity.GetRelation    <AttackRelation,Entity>(entity2); // O(1)
```

Methods to mutate and query link relation are at
[RelationExtensions](https://github.com/friflo/Friflo.Engine-docs/blob/main/api/RelationExtensions.md).

The following example illustrate the state changes by small text graphs on the right.  
Link relations are represented by `→` icon.  
This example show the graph changes when adding link relations or deleting entities.

```cs
struct AttackRelation : ILinkRelation {
    public  Entity  target;
    public  Entity  GetRelationKey() => target;
}

public static void LinkRelations()
{
    var store   = new EntityStore();
    
    var entity1 = store.CreateEntity(1);                          // link relations
    var entity2 = store.CreateEntity(2);                          // symbolized as →
    var entity3 = store.CreateEntity(3);                          //   1     2     3
    
    // add a link relation to entity (2) referencing entity (1)
    entity2.AddRelation(new AttackRelation { target = entity1 }); //   1  ←  2     3
    // get all links added to the entity. O(1)
    entity2.GetRelations    <AttackRelation>();     // { 1 }
    // get all incoming links. O(1)
    entity1.GetIncomingLinks<AttackRelation>();     // { 2 }
    
    // add another one. An entity can have multiple link relations
    entity2.AddRelation(new AttackRelation { target = entity3 }); //   1  ←  2  →  3
    entity2.GetRelations    <AttackRelation>();     // { 1, 3 }
    entity3.GetIncomingLinks<AttackRelation>();     // { 2 }
    
    // deleting a linked entity (1) removes all link relations referencing it
    entity1.DeleteEntity();                                       //         2  →  3
    entity2.GetRelations    <AttackRelation>();     // { 3 }
    
    // deleting entity (2) is reflected by incoming links query
    entity2.DeleteEntity();                                       //               3
    entity3.GetIncomingLinks<AttackRelation>();     // { }
}
```

<br/>


# Relations

> [!IMPORTANT]
> Breaking changes with `3.0.0-preview.16`
>
> 1. To avoid mixing up relations with components accidentally `IRelation` does not extends `IComponent` anymore.
> 2. Renamed public API's  
     `IRelationComponent<>` -> `IRelation<>`  
     `RelationComponents<>` -> `Relations<>`



A relation enables adding multiple components of the same type to an entity.

A typical limitation of an archetype-based ECS is that an entity can only contain one component of a certain type.  
When adding a component of a specific type to an entity already present the component is updated.  
This is the common behavior implemented by most ECS implementations like **EnTT**, **flecs**, **BEVY**, **fenn**ecs, ... .  
An alternative approach is allowing undefined behavior like **Arch** resulting in memory corruption, access violation, ... .

A relation in mathematical context describes a connection between elements of two sets.  
Transferred to this ECS: The first set are **all entities** the second set are all possible **relation keys**.

## Relation

A relation that can be added to an entity multiple times must implement
[`IRelation<>`](https://github.com/friflo/Friflo.Engine-docs/blob/main/api/IRelation_TKey_.md).  
To distinguish between different relations, a relation type must implement `GetRelationKey()`.  
Any type can be used as relation key type. E.g. enum, string, int, long, Guid, DateTime, ... .

Relations are added, removed and queried with
```cs
    entity.AddRelation(new InventoryItem { type = ItemType.Coin });
    entity.AddRelation(new InventoryItem { type = ItemType.Axe  });
    entity.RemoveRelation <InventoryItem,ItemType>(ItemType.Coin);
    entity.GetRelations   <InventoryItem>();                        // O(1)
    entity.GetRelation    <InventoryItem,ItemType>(ItemType.Axe);   // O(1)
```

The following example illustrates the state changes of a specific entity regarding its relations.  
It uses an `enum` as relation key type.

```cs
enum ItemType {
    Coin    = 1,
    Axe     = 2,
}

struct InventoryItem : IRelation<ItemType> { // relation key type: ItemType
    public  ItemType    type;
    public  int         count;
    public  ItemType    GetRelationKey() => type;     // unique relation key
}

public static void Relations()
{
    var store   = new EntityStore();
    var entity  = store.CreateEntity();
    
    // add multiple relations of the same component type
    entity.AddRelation(new InventoryItem { type = ItemType.Coin, count = 42 });
    entity.AddRelation(new InventoryItem { type = ItemType.Axe,  count =  3 });
    
    // Get all relations added to an entity.   O(1)
    entity.GetRelations  <InventoryItem>();                       // { Coin, Axe }
    
    // Get a specific relation from an entity. O(1)
    entity.GetRelation   <InventoryItem,ItemType>(ItemType.Coin); // {type=Coin, count=42}
    
    // Remove a specific relation from an entity
    entity.RemoveRelation<InventoryItem,ItemType>(ItemType.Axe);
    entity.GetRelations  <InventoryItem>();                       // { Coin }
}
```

## Serialization

Serialization of relations as JSON is supported since `3.0.0-preview.17` or higher.  
Its encoding similar to the serialization of components.  
In contrast to components relations are serialized as an array of components as a single entity can have multiple relations.

```json
{
    "id": 42,
    "components": {
        "relations": [{"value":42},{"value":43}],
        "component": {"value":42}
    }
}
```



<br/>


# Search

This ECS enables efficient search of indexed component fields.  
This enables **full-text search** by using `string` as the indexed component type like in the example below.  
Any type can be used as indexed component type. E.g. int, long, float, Guid, DateTime, enum, ... .  
A search / query for a specific value executes in O(1).


## Indexed Component

An [IIndexedComponent<>](https://github.com/friflo/Friflo.Engine-docs/blob/main/api/IIndexedComponent_TValue_.md)
provide the same functionality and behavior as normal components implementing `IComponent`.  
Indexing is implement using an [inverted index ⋅ Wikipedia](https://en.wikipedia.org/wiki/Inverted_index).  
Adding, removing or updating an indexed component updates the index.  
These operations are executed in O(1) but significant slower than the non indexed counterparts ~10x.  
*Performance:* Indexing 1000 different component values ~60 μs.

**Range query**

In case the indexed component type implements `IComparable<>` like int, string, DateTime, ... range queries can be executed.  
A range query returns all entities with a component value in the specified range. See example code.

Methods to query indexed components are at [IndexExtensions](https://github.com/friflo/Friflo.Engine-docs/blob/main/api/IndexExtensions.md).

```cs
struct Player : IIndexedComponent<string>       // indexed field type: string
{
    public  string  name;
    public  string  GetIndexedValue() => name;  // indexed field
}

public static void IndexedComponents()
{
    var store   = new EntityStore();
    var index   = store.ComponentIndex<Player,string>();
    for (int n = 0; n < 1000; n++) {
        var entity = store.CreateEntity();
        entity.AddComponent(new Player { name = $"Player-{n,0:000}"});
    }
    // get all entities where Player.name == "Player-001". O(1)
    var entities = index["Player-001"];                                    // Count: 1
    
    // return same result as lookup using a Query(). O(1)
    store.Query().HasValue    <Player,string>("Player-001");               // Count: 1
    
    // return all entities with a Player.name in the given range.
    // O(N ⋅ log N) - N: all unique player names
    store.Query().ValueInRange<Player,string>("Player-000", "Player-099"); // Count: 100
    
    // get all unique Player.name's. O(1)
    var values = index.Values;                                             // Count: 1000
}
```

<br/>


# Performance

All specialized component types are optimized for performance and low memory footprint.  
Read and write access to those components are executed without boxing.  
Each component type requires a single dictionary and up to a dozen arrays, regardless of the number of components stored.    
E.g. one, one thousand or one million  components.  
This make these component types as friendly for garbage collection as general components.  
If the array buffers are large enough, there are no further memory allocations.

## Benchmark - Relations

The number of link relations / relations per entity should not exceed 100.  
The reason is that inserting and removing a relation is executed in O(N).  
N: number of relations per entity.

*Benchmark* [see Tests](https://github.com/friflo/Friflo.Engine.ECS/blob/main/src/Tests/ECS/Relations/Test_Relations_Query.cs)  
Add 1.000.000 int relations. Type:
```cs
internal struct IntRelation : IRelation<int> {
    public          int     value;
    public          int     GetRelationKey()    => value;
}
```
| relationsPerEntity |        duration ms |           entities |
| ------------------:| ------------------:| ------------------:|
|                  1 |              60.17 |            1000000 |
|                  2 |              64.11 |             500000 |
|                  4 |              62.68 |             250000 |
|                  8 |              66.66 |             125000 |
|                 16 |              67.71 |              62500 |
|                 32 |              90.53 |              31250 |
|                 64 |             150.14 |              15625 |
|                128 |             154.39 |               7813 |
|                256 |             254.26 |               3907 |
|                512 |             416.20 |               1954 |
|               1024 |             772.39 |                977 |
|               2048 |            1504.50 |                489 |
|               4096 |            2993.34 |                245 |
|               8192 |            5867.59 |                123 |


## Benchmark - Indexing

The number of components having the same key value should not exceed 100.  
The reason is that inserting and removing a component to / from the index is executed in O(N).  
N: number of components having the same key value (duplicates).

*Benchmark* [see Tests](https://github.com/friflo/Friflo.Engine.ECS/blob/main/src/Tests/ECS/Index/Test_Index.cs)  
Add 1.000.000 indexed int components. Each to an individual entity. Type:
```cs
public struct IndexedInt : IIndexedComponent<int> {
    public          int         value;
    public          int         GetIndexedValue()   => value;
}
```
|     duplicateCount |        duration ms |           entities |
| ------------------:| ------------------:| ------------------:|
|                  1 |             119.68 |            1000000 |
|                  2 |              91.73 |            1000000 |
|                  4 |              96.08 |            1000000 |
|                  8 |              97.06 |            1000000 |
|                 16 |             103.75 |            1000000 |
|                 32 |             150.19 |            1000000 |
|                 64 |              98.52 |            1000000 |
|                128 |             106.64 |            1000000 |
|                256 |             111.62 |            1000000 |
|                512 |             123.53 |            1000000 |
|               1024 |             137.13 |            1000000 |
|               2048 |             254.83 |            1000000 |
|               4096 |             234.58 |            1000000 |
|               8192 |             433.60 |            1000000 |
|              16384 |             817.92 |            1000000 |
|              32768 |            1662.16 |            1000000 |

