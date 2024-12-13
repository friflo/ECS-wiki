The `Entity` is the main data structure when working with an ECS.  

An `Entity` has a unique identity - `Id` - and acts as a container for components, tags, script and child entities.  
An `EntityStore` is a container of entities and used to create entities with `store.CreateEntity()`.

```csharp
public static void CreateEntity()
{
    var store = new EntityStore();
    store.CreateEntity();
    store.CreateEntity();
    
    foreach (var entity in store.Entities) {
        Console.WriteLine($"entity {entity}");
    }
    // > entity id: 1  []       Info:  [] entity has no components
    // > entity id: 2  []
}
```

Entities can be deleted with `entity.DeleteEntity()`.  
Variables of type `Entity` mimic the behavior of reference types.  
Using an entity method on a deleted entity throws a `NullReferenceException`.  
To handled this case use `entity.IsNull`.

```csharp
public static void DeleteEntity()
{
    var store   = new EntityStore();
    var entity  = store.CreateEntity();
    entity.DeleteEntity();
    var isDeleted = entity.IsNull;
    Console.WriteLine($"deleted: {isDeleted}");     // > deleted: True
}
```

Entities can be disabled.  
Disabled entities are excluded from query results by default.  
To include disabled entities in a query result use `query.WithDisabled()`.

```csharp
public static void DisableEntity()
{
    var store   = new EntityStore();
    var entity  = store.CreateEntity();
    entity.Enabled = false;
    Console.WriteLine(entity);                      // > id: 1  [#Disabled]
    
    var query    = store.Query();
    Console.WriteLine($"default - {query}");        // > default - Query: []  Count: 0
    
    var disabled = store.Query().WithDisabled();
    Console.WriteLine($"disabled - {disabled}");    // > disabled - Query: []  Count: 1
}
```

Entity example code is part of the unit tests see:
[Tests/ECS/Examples](https://github.com/friflo/Friflo.Engine.ECS/tree/main/src/Tests/ECS/Examples).

When tying out the examples use a debugger to check entity state changes while stepping throw the code.

<img src="../images/entity-debugger.png" width="630" height="370"></img>  
*Screenshot:* Entity state - enables browsing the entire store hierarchy.

Examples showing typical use cases of the [Entity API](https://github.com/friflo/Friflo.Engine-docs/blob/main/api/Entity.md)


## EntityStore

An `EntityStore` is a container for entities running as an in-memory database.  
It is highly optimized for efficient storage fast queries and event handling.  
In other ECS implementations this type is typically called *World*.

The store enables to
- create entities
- modify entities - add / remove components, tags, scripts and child entities
- query entities with a specific set of components or tags
- subscribe events like adding / removing components, tags, scripts and child entities

Multiple stores can be used in parallel and act completely independent from each other.  
The example shows how to create a store. Mainly every example will start with this line.

```csharp
public static void CreateStore()
{
    var store = new EntityStore();
}
```
<br/>


## Component

`Components` are `struct`s used to store data on entities.  
Multiple components with different types can be added / removed to / from an entity.  
If adding a component using a type already stored in the entity its value gets updated. 

```csharp
[ComponentKey("my-component")]
public struct MyComponent : IComponent {
    public int value;
}

public static void AddComponents()
{
    var store   = new EntityStore();
    var entity  = store.CreateEntity();
    
    // add components
    entity.AddComponent(new EntityName("Hello World!"));// EntityName is build-in
    entity.AddComponent(new MyComponent { value = 42 });
    Console.WriteLine($"entity: {entity}");             // > entity: id: 1  "Hello World!"  [EntityName, Position]
    
    // get component
    Console.WriteLine($"name: {entity.Name.value}");    // > name: Hello World!
    var value = entity.GetComponent<MyComponent>().value;
    Console.WriteLine($"MyComponent: {value}");         // > MyComponent: 42
    
    // Serialize entity to JSON
    Console.WriteLine(entity.DebugJSON);
}
```

Result of `entity.DebugJSON`:
```json
{
    "id": 1,
    "components": {
        "name": {"value":"Hello World!"},
        "my-component": {"value":42}
    }
}
```
<br/>


## Unique entity

Add a `UniqueEntity` component to an entity to mark it as a *"singleton"* with a unique `string` id.  
The entity can than be retrieved with `EntityStore.GetUniqueEntity()` to reduce code coupling.  
It enables access to a unique entity without the need to pass an entity by external code.   

```csharp
public static void GetUniqueEntity()
{
    var store   = new EntityStore();
    store.CreateEntity(new UniqueEntity("Player"));     // UniqueEntity is build-in
    
    var player  = store.GetUniqueEntity("Player");
    Console.WriteLine($"entity: {player}");             // > entity: id: 1  [UniqueEntity]
}
```
<br/>

> **Info**  
> Since version 3.0.0 there is more flexible and performant alternative available by using a [Component Index](/docs/component-index.md).  
> It supports defining a custom `IIndexedComponent<>` type and have several advantages:
> - The unique key can be of any type - e.g. `int`, `Guid`, `enum`, `string`, ... . The key of unique entities is always a `string`.
> - The storage of a **Component Index** is optimized for low memory footprint and fast lookup.
> - Additional fields can be added to an `IIndexedComponent<>` type.


## Tag

`Tags` are `struct`s similar to components - except they store no data.  
They can be utilized in queries similar as components to restrict the amount of entities returned by a query.  
If adding a tag using a type already attached to the entity the entity remains unchanged.

```csharp
public struct MyTag1 : ITag { }
public struct MyTag2 : ITag { }

public static void AddTags()
{
    var store   = new EntityStore();
    var entity  = store.CreateEntity();
    
    // add tags
    entity.AddTag<MyTag1>();
    entity.AddTag<MyTag2>();
    Console.WriteLine($"entity: {entity}");     // > entity: id: 1  [#MyTag1, #MyTag2]
    
    // get tag
    var tag1 = entity.Tags.Has<MyTag1>();
    Console.WriteLine($"tag1: {tag1}");         // > tag1: True
}
```
<br/>


## Hierarchy

A typical use case in an Game or Editor is to build up a hierarchy of entities.  
To add an entity as a child to another entity use `Entity.AddChild()`.  
In case the added child already has a parent it gets removed from the old parent.  
The children of the added (moved) entity remain being its children.  
If removing a child from its parent all its children are removed from the hierarchy.

```csharp
public static void AddChildEntities()
{
    var store   = new EntityStore();
    var root    = store.CreateEntity();
    var child1  = store.CreateEntity();
    var child2  = store.CreateEntity();
    
    // add child entities
    root.AddChild(child1);
    root.AddChild(child2);
    
    Console.WriteLine($"entities: {root.ChildEntities}"); // > entities: Count: 2
}
```
<br/>


## Archetype

An `Archetype` defines a specific set of components and tags for its entities.  
At the same time it is also a container of entities with exactly this combination of components and tags.  

> **Explanation**  
> An `Archetype` instance corresponds to an SQL `TABLE` where its components are the counterpart of the table rows.  
> In contrast to tables archetypes are created automatically on demand. A relational database requires to create tables upfront.

The following comparison shows the difference in modeling types in **ECS** vs **OOP**.  

<table>
<tr>
<th>ECS - Composition</th>
<th>OOP - Polymorphism</th>
</tr>
<tr>
  <td><i>Inheritance</i><br/>
      ECS does not utilize inheritance.<br/>
      It prefers composition over inheritance.
  </td>
  <td><br/>
      Common OPP is based on inheritance.<br/>
      Likely result: A god base class responsible for everything. 😊
  </td>  
</tr>
<tr>
  <td><i>Code coupling</i><br/>
      Data lives in components - behavior in systems.<br/>
      New behaviors does not affect existing code.
  </td>
  <td><br/>
      Data and behavior are both in classes.<br/>
      New behaviors may add dependencies or side effects.
  </td>  
</tr>
<tr>
  <td><i>Storage</i><br/>
      An Archetype is also a container of entities.
  </td>
  <td><br/>
      Organizing containers is part of application code.
  </td>
</tr>
<tr>
  <td><i>Changing a type</i><br/>
      Supported by adding/removing tags or components.
  </td>
  <td><br/>
      Type is fixed an cannot be changed.
  </td>
</tr>
<tr>
  <td><i>Component access / visibility</i><br/>
    Having a reference to an EntityStore enables<br/>
    unrestricted reading and changing of components.
  </td>
  <td><br/>
    Is controlled by access modifiers:<br/>
    public, protected, internal and private.
  </td>
</tr>

<tr>
  <td colspan="2" align="center"><b>Example</b>
  </td>
</tr>

<tr>
<td style="padding:0px;">

```csharp
// No base class Animal in ECS
struct Dog : ITag { }
struct Cat : ITag { }


var store = new EntityStore();

var dogType = store.GetArchetype(Tags.Get<Dog>());
var catType = store.GetArchetype(Tags.Get<Cat>());
WriteLine(dogType.Name);            // [#Dog]

dogType.CreateEntity();
catType.CreateEntity();

var dogs = store.Query().AnyTags(Tags.Get<Dog>());
var all  = store.Query().AnyTags(Tags.Get<Dog, Cat>());

WriteLine($"dogs: {dogs.Count}");   // dogs: 1
WriteLine($"all: {all.Count}");     // all: 2
```

</td>
<td style="padding:0px;">

```csharp
class Animal { }
class Dog : Animal { }
class Cat : Animal { }


var animals = new List<Animal>();

var dogType = typeof(Dog);
var catType = typeof(Cat);
WriteLine(dogType.Name);            // Dog

animals.Add(new Dog());
animals.Add(new Cat());

var dogs = animals.Where(a => a is Dog);
var all  = animals.Where(a => a is Dog or Cat);

WriteLine($"dogs: {dogs.Count()}"); // dogs: 1
WriteLine($"all: {all.Count()}");   // all: 2
```

</td>
</tr>

<tr>
  <td colspan="2" align="center"><b>Performance</b>
  </td>
</tr>
<tr>
  <td><i>Runtime complexity O() of queries for specific types</i><br/>
      O(size of result set)
  </td>
  <td><br/>
      O(size of all objects)
  </td>
</tr>
<tr>
  <td><i>Memory layout</i><br/>
      Continuous memory in heap - high hit rate of L1 cache.
  </td>
  <td><br/>
      Randomly placed in heap - high rate of L1 cache misses.
  </td>
</tr>
<tr>
  <td><i>Instruction pipelining</i><br/>
      Minimize conditional branches in update loops.<br/>
      Process multiple components at once using SIMD.
  </td>
  <td><br/><br/>
      Virtual method calls prevent branch prediction.
  </td>
</tr>

</table>
<br/>


## JSON Serialization

The entities stored in an EntityStore can be serialized as JSON using an
[EntitySerializer](https://github.com/friflo/Friflo.Engine-docs/blob/main/api/EntitySerializer.md).  


> **Note**  
> Currently serialization / deserialization only support struct fields - properties not.
> See [Issue #28](https://github.com/friflo/Friflo.Engine.ECS/issues/28).  
> Component and relation types are required to be struct's.

Writing the entities of a store to a JSON file is done with `WriteStore()`.  
Reading the entities of a JSON file into a store with `ReadIntoStore()`.

```csharp
public static void JsonSerialization()
{
    var store = new EntityStore();
    store.CreateEntity(new EntityName("hello JSON"));
    store.CreateEntity(new Position(1, 2, 3));

    // --- Write store entities as JSON array
    var serializer = new EntitySerializer();
    var writeStream = new FileStream("entity-store.json", FileMode.Create);
    serializer.WriteStore(store, writeStream);
    writeStream.Close();
    
    // --- Read JSON array into new store
    var targetStore = new EntityStore();
    serializer.ReadIntoStore(targetStore, new FileStream("entity-store.json", FileMode.Open));
    
    Console.WriteLine($"entities: {targetStore.Count}"); // > entities: 2
}
```

The JSON content of the file `"entity-store.json"` created with `serializer.WriteStore()`
```json
[{
    "id": 1,
    "components": {
        "name": {"value":"hello JSON"}
    }
},{
    "id": 2,
    "components": {
        "pos": {"x":1,"y":2,"z":3}
    }
}]
```
<br/>