
A **ComponentIndex** enables efficient search of indexed component field values.  
This enables **lookup** aka **search** of entities having components with specific a specific value like in the example below.  
Any type can be used as indexed component type. E.g. `int`, `long`, `float`, `Guid`, `DateTime`, `enum`, ... .  
A search / query for a specific value executes in O(1).

## Indexed Components

An [IIndexedComponent<>](https://github.com/friflo/Friflo.Engine-docs/blob/main/api/IIndexedComponent_TValue_.md)
provide the same functionality and behavior as normal components implementing `IComponent`.  
For every indexed component type the store creates a an [inverted index ⋅ Wikipedia](https://en.wikipedia.org/wiki/Inverted_index).  
Adding, removing or updating an indexed component updates the index.  
These operations are executed in O(1) but significant slower than the non indexed counterparts ~10x.  
*Performance:* Indexing 1000 different component values ~60 μs.

```cs
struct TileComponent : IIndexedComponent<int> // indexed field type: int
{
    public  int  tileId;
    public  int  GetIndexedValue() => tileId; // indexed field
}

public static void ComponentLookup()
{
    var store   = new EntityStore();
    var index   = store.ComponentIndex<TileComponent,int>();
    store.CreateEntity(new TileComponent{ tileId = 10 });
    store.CreateEntity(new TileComponent{ tileId = 20 });

    // lookup entities where component tileId = 10 in O(1)
    var entities = index[10];   // Count: 1

    // get all unique tileId's in O(1)
    var values = index.Values;  // { 10, 20 }
}
```

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