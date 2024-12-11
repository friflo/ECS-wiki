## Entity events

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


## EventRecorder