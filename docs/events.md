Events are messages sent used to notify about state changes on an `Entity`.

These events can be consumed in two different ways.
- Process events directly by an event handler subscribed to an event like `entity.OnComponentChanged`.
- Or by recording all events using an [EventRecorder](#eventrecorder) and process recorded events later within a `Query`.

## Entity changes

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
<br/>


## EventRecorder

An `EventFilter` is used to process events recorded by an `EventRecorder`.  
`EventFilter`'s can be used on its own or within a query like in the example below.  
All events that need to be filtered - like added/removed components/tags - can be added to the `EventFilter`.  
E.g. `ComponentAdded<Position>()` or `TagAdded<MyTag1>`.  

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

<br/>


## Signals

`Signal`s are similar to events. They are used to send and receive custom events on entity level in an application.  
They have the same characteristics as events described in the section above.  
The use of `Signal`'s is intended for scenarios when something happens occasionally.  
This avoids the need to check a state every frame.

```csharp
public readonly struct MySignal { } 

public static void AddSignalHandler()
{
    var store   = new EntityStore();
    var entity  = store.CreateEntity();
    entity.AddSignalHandler<MySignal>(signal => { Console.WriteLine(signal); }); // > entity: 1 - signal > MySignal    
    entity.EmitSignal(new MySignal());
}
```
<br/>

