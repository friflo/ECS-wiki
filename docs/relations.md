A relation enables adding multiple *"components"* of the same type to an entity.  

> *Info*  
> Relations are not components but both have similar interfaces too add, remove or access them.  
> Relations are not stored within archetypes to avoid archetype fragmentation.

A typical limitation of an archetype-based ECS is that an entity can only contain one component of a certain type.  
When adding a component of a specific type to an entity already present the component is updated.  
This is the common behavior implemented by most ECS implementations like **EnTT**, **flecs**, **BEVY**, **fenn**ecs, ... .  

**Terminology**  
A relation in mathematical context describes a connection between the elements of two sets: *Set-1* & *Set-2*.  
In other words - a relation is a pair (element of *Set-1*, element of *Set-2*).

In friflo ECS a relation is a type implementing either `IRelation<>` or `IRelationLink<>`.  
An entity containing a relation creates a relation between this entity and the relation key.  
So *Set-1* are always **all entities** and *Set-2* are all possible **relation keys**.  

- A **relation** implements `IRelation<>` and is a pair (entity, relation key)
- A **relationship** implements `IRelationLink<>` and is a pair (entity, linked entity)

## Relation

Multiple relations can be added to a single entity and must implement
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

> **Remarks**  
> Breaking changes since `3.0.0-preview.16`
>
> 1. To avoid mixing up relations with components accidentally `IRelation` does not extends `IComponent` anymore.
> 2. Renamed public API's  
     `IRelationComponent<>` -> `IRelation<>`  
     `RelationComponents<>` -> `Relations<>`

<br/>


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