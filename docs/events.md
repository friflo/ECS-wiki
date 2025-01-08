Events are messages sent used to notify about state changes of an `Entity`.

These events can be consumed in two different ways.
- Process events directly by an event handler subscribed to an event like `entity.OnComponentChanged`.
- Or by recording all events using an [EventRecorder](#eventrecorder) and process recorded events later within a `Query`.

# Entity changes

If changing an entity by adding or removing components, tags, scripts or child entities events are emitted.  
An application can subscribe to these events like shown in the example.  
Emitting these type of events increase code decoupling.  
Without events these modifications need to be notified by direct method calls.  
The *build-in* events can be subscribed on `EntityStore` and on `Entity` level like shown in the example below.  

```csharp
public static void AddEventHandlers()
{
    var store   = new EntityStore();
    var entity  = store.CreateEntity();
    entity.OnComponentChanged     += ev => { Console.WriteLine(ev); }; // > entity: 1 - event > Add Component: [MyComponent]
    entity.OnTagsChanged          += ev => { Console.WriteLine(ev); }; // > entity: 1 - event > Add Tags: [#MyTag1]
    entity.OnScriptChanged        += ev => { Console.WriteLine(ev); }; // > entity: 1 - event > Add Script: [*MyScript]
    entity.OnChildEntitiesChanged += ev => { Console.WriteLine(ev); }; // > entity: 1 - event > Add Child[0] = 2

    entity.AddComponent(new MyComponent());
    entity.AddTag<MyTag1>();
    entity.AddScript(new MyScript());
    entity.AddChild(store.CreateEntity());
}
```

## Component events

Event handlers for component changes notify one of the following `ev.Action`
- `Add` - a component was newly added to the entity.
- `Update` - the component value changed.
- `Remove` - the component was removed.

In case of `Update` or `Remove` the handler provide access to the old component value.

```csharp
public static void ComponentEvents()
{
    var store  = new EntityStore();
    var entity = store.CreateEntity();
    entity.OnComponentChanged += ev =>
    {
        if (ev.Type == typeof(EntityName)) {
            string log = ev.Action switch
            {
                Add    => $"new: {ev.Component<EntityName>()}",
                Update => $"new: {ev.Component<EntityName>()}  old: {ev.OldComponent<EntityName>()}",
                Remove => $"old: {ev.OldComponent<EntityName>()}",
                _      => null
            };
            Console.WriteLine($"{ev.Action} {log}");
        }
    };
    entity.AddComponent(new EntityName("Peter"));
    entity.AddComponent(new EntityName("Paul"));
    entity.RemoveComponent<EntityName>();
}
```

Log Output 
```js
Add new: 'Peter'
Update new: 'Paul'  old: 'Peter'
Remove old: 'Paul
```

## Tag events

Event handlers for tag changes notify which entity tags are changed.
- `AddedTags` - tags added to an entity.
- `RemovedTags` - tags removed from an entity.
- `ChangedTags` - tags added or removed from an entity.

```csharp
public struct MyTag1 : ITag { }
public struct MyTag2 : ITag { }

public static void TagEvents()
{
    var store  = new EntityStore();
    var entity = store.CreateEntity();
    entity.OnTagsChanged += ev =>
    {
        string log = "";
        if (ev.AddedTags.  Has<MyTag1>()) { log += ", added:   MyTag1"; }
        if (ev.RemovedTags.Has<MyTag1>()) { log += ", removed: MyTag1"; }
        
        if (ev.AddedTags.  Has<MyTag2>()) { log += ", added:   MyTag2"; }
        if (ev.RemovedTags.Has<MyTag2>()) { log += ", removed: MyTag2"; }
        
        Console.WriteLine($"entity {entity.Id}{log}");
    };
    entity.AddTag<MyTag1>();
    entity.RemoveTag<MyTag1>();
    entity.AddTags(Tags.Get<MyTag1, MyTag2>());
}
```

Log Output 
```js
entity 1, added:   MyTag1
entity 1, removed: MyTag1
entity 1, added:   MyTag1, added:   MyTag2
```
<br/>


# EventRecorder

An `EventRecorder` record all component and tag changes of an `EntityStore` when `Enabled`.  
A recorder is required for queries using `EventFilter`'s.  
To clear all recorded events use `store.EventRecorder.ClearEvents()`.  
In a game loop this is typically performed at the beginning of a new frame.  
To stop recording events entirely use `store.EventRecorder.Enabled = false`.

## EventFilter

The intended use-case for `EventFilter`'s are queries.  
When iterating the entities of a query result it can be checked if an entity was changed by an operation matching the specified filters.  
E.g a specific component or tag was added.
```cs
    query.EventFilter.ComponentAdded<Position>();
    query.EventFilter.TagAdded<MyTag1>();
```

`EventFilter`'s can be used on its own or within a query, see the examples below.  
All events that need to be filtered - like added/removed components/tags - can be added to the `EventFilter`.  

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

```csharp
public static void FilterEventsQuery()
{
        var store = new EntityStore();
        store.EventRecorder.Enabled = true; // required for EventFilter

        store.CreateEntity(new Position(), Tags.Get<MyTag1>());

        var query = store.Query<Position, MyTag1>();
        query.EventFilter.ComponentAdded<Position>();
        query.EventFilter.TagAdded<MyTag1>();

        query.ForEachEntity((ref Position, Entity entity) =>
        {
            bool hasEvent = query.HasEvent(entity.Id);
            Console.WriteLine($"{entity} - hasEvent: {hasEvent}");
        });

        // > [Position] - hasEvent: True
}
```

<br/>


# Signals

**Signals** are similar to events.  
They are used to **send** and **subscribe** **custom events** on entity level in an application.  
To prevent heap allocations signal types must be structs.  

The use of signals is intended for scenarios when something happens occasionally.  
This avoids the need to check a state every frame to detect a specific condition.  
For example signals could be used to react on collisions between entities.

Signal handlers can be added as shown below and remove if needed.
```csharp
    var handler = entity.AddSignalHandler<MyEvent2>(signal => { ... });
    entity.RemoveSignalHandler(handler);
```
Multiple signal handlers can be added to an entity and are automatically removed if the entity is deleted.

```csharp
public struct CollisionSignal {
    public Entity other;
}

public static void AddSignalHandler()
{
    var store  = new EntityStore();
    var player = store.CreateEntity(1);
    player.AddSignalHandler<CollisionSignal>(signal => {
        Console.WriteLine($"player collision with entity: {signal.Event.other.Id}");
        // > player collision with entity: 2
    });
    var npc = store.CreateEntity(2);
    // ... detect collisions with a collision system. In case of collision:
    player.EmitSignal(new CollisionSignal{ other = npc });
}
```

> **Note** Avoid emitting signals inside a query loop.

When using a query loop to detect collisions signals should not be emitted directly.  
The the event handler may perform a structural change - e.g removing or adding a components.  
Doing this will invalidate the query loop.  
To avoid this detected collisions can be stored inside the query loop in a `List<>`.  
After the collision loop finishes collision events can me emitted and are able to perform structural changes.

<br/>

