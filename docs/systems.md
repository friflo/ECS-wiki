Systems in an ECS are typically queries.  
So you can still use the `world.Query<Position, Velocity>()` shown in the "Hello World" example.  

Using Systems is optional but they have some significant advantages.

**System features**

- Enable chaining multiple decoupled [QuerySystem](https://github.com/friflo/Friflo.Engine-docs/blob/main/api/QuerySystem.md) classes in a
  [SystemGroup](https://github.com/friflo/Friflo.Engine-docs/blob/main/api/SystemGroup.md).  
  Each group provide a [CommandBuffer](https://friflo.gitbook.io/friflo.engine.ecs/examples/optimization#commandbuffer).

- A system can have state - fields or properties - which can be used as parameters in `OnUpdate()`.  
  The system state can be serialized to JSON.

- Systems can be enabled/disabled or removed.  
  The order of systems in a group can be changed.

- Systems have performance monitoring build-in to measure execution times and memory allocations.  
  If enabled systems detected as bottleneck can be optimized.  
  A perf log (see example below) provide a clear overview of all systems their amount of entities and impact on performance.

- Multiple worlds can be added to a single  [SystemRoot](https://github.com/friflo/Friflo.Engine-docs/blob/main/api/SystemRoot.md) instance.  
  `root.Update()` will execute every system on all worlds.


```csharp
public static void HelloSystem()
{
    var world = new EntityStore();
    for (int n = 0; n < 10; n++) {
        world.CreateEntity(new Position(n, 0, 0), new Velocity(), new Scale3());
    }
    var root = new SystemRoot(world) {
        new MoveSystem(),
    //  new PulseSystem(),
    //  new ... multiple systems can be added. The execution order still remains clear.
    };
    root.Update(default);
}
        
class MoveSystem : QuerySystem<Position, Velocity>
{
    protected override void OnUpdate() {
        Query.ForEachEntity((ref Position position, ref Velocity velocity, Entity entity) => {
            position.value += velocity.value;
        });
    }
}
```

A valuable strength of an ECS is establishing a clear and decoupled code structure.  
Adding the `PulseSystem` below to the `SystemRoot` above is trivial.  
This system uses a `foreach (var entity in Query.Entities)` as an alternative to `Query.ForEachEntity((...) => {...})`  
to iterate the query result.

```csharp
struct Pulsating : ITag { }

class PulseSystem : QuerySystem<Scale3>
{
    float frequency = 4f;
    
    public PulseSystem() => Filter.AnyTags(Tags.Get<Pulsating>());
    
    protected override void OnUpdate() {
        foreach (var entity in Query.Entities) {
            ref var scale = ref entity.GetComponent<Scale3>().value;
            scale = Vector3.One * (1 + 0.2f * MathF.Sin(frequency * Tick.time));
        }
    }
}
```

### ⏱ System monitoring

System performance monitoring is disabled by default.  
To enable monitoring call:

```csharp
root.SetMonitorPerf(true);
```

When enabled system monitoring captures
- Number of system executions.
- System execution duration in ms.
- Memory heap allocations per system in bytes.
- The number of entities matching a query system.


#### Realtime monitoring

In a game editor like Unity system monitoring is available in the **ECS System Set** component.
<details>
<summary>Screenshot: <b>ECS System Set</b> component in Play mode</summary>
<img src="https://raw.githubusercontent.com/friflo/Friflo.Engine.ECS/main/docs/images/SystemSet-Unity.png" width="324" height="ddd"/>
</details>


#### Log monitoring

The performance statistics available at [SystemPerf](https://github.com/friflo/Friflo.Engine-docs/blob/main/api/SystemPerf.md).  
To get performance statistics on console use:

```csharp
root.Update(default);
Console.WriteLine(root.GetPerfLog());
```

The log result will look like:
```js
stores: 1                  on   last ms    sum ms   updates  last mem   sum mem  entities
---------------------      --  --------  --------  --------  --------  --------  --------
Systems [2]                 +     0.076     3.322        10       128      1392
| ScaleSystem               +     0.038     2.088        10        64       696     10000
| PositionSystem            +     0.038     1.222        10        64       696     10000
```
```
on                  + enabled  - disabled
last ms, sum ms     last/sum system execution time in ms
updates             number of executions
last mem, sum mem   last/sum allocated bytes
entities            number of entities matching a QuerySystem
```