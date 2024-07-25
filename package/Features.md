
## Integration

Friflo.Engine.ECS enables you integration in any C# based Game Engine and any .NET enterprise application.

- **Unity** - ![new](../images/new.svg) - Integration as nuget package. Tested 2022.3.20f1 (Mono & AOT/IL2CPP).  
    Use [NuGetForUnity](https://github.com/GlitchEnzo/NuGetForUnity) to install nuget package **Friflo.Engine.ECS**. 1.23.0 or higher.
    Usage in [Unity script example](https://friflo.gitbook.io/ecs-wiki/Examples-General#unity-update-script).
- **.NET** - Supported target frameworks: **.NET Standard 2.1 .NET 5 .NET 6 .NET 7 .NET 8**. Supports **WASM / WebAssembly**.  
    Integration into a .NET project as nuget package see [nuget ⋅ Friflo.Engine.ECS](https://www.nuget.org/packages/Friflo.Engine.ECS/).  
- **Godot** - Integration as nuget package. Tested with Godot 4.1.1.


## Performance

You want best ECS performance?  
You get it! 😊  
Without using unsafe code! 🔒

E.g Creating 1.000.000 entities with two components: 13 ms.  
Best ECS C++ implementation **pico_ecs**: 45 ms  
See: [C++ ECS Benchmarks ⋅ Create Entities](https://github.com/abeimler/ecs_benchmark?tab=readme-ov-file#create-entities)


- Uses array buffers and cache query instances -> no memory allocations after buffers are large enough.
- High memory locality by storing components in continuous memory.
- Optimized for high L1 cache line hit rate.
- Best benchmark results at: [**Ecs.CSharp.Benchmark - GitHub**](https://github.com/Doraku/Ecs.CSharp.Benchmark).
- Processing components of large queries has the memory bandwidth as bottleneck. Either using multi threading or SIMD.  
    Alternative ECS implementations using C/C++, Rust, Zig or Mojo 🔥 cannot be faster due to the physical limits.


## ECS

- Developer friendly / OOP like API by exposing the [Entity API](https://github.com/friflo/Friflo.Engine-docs/blob/main/api/Entity.md)
  **struct** as the main interface.  
  Or compare the `Entity` API with other API's at [Engine-comparison.md](https://github.com/friflo/Friflo.Json.Fliox/blob/main/Engine/docs/Engine-comparison.md).  
  The typical alternative of an ECS implementations is providing a `World` class and using `int` parameters as entity `id`s.
- Record entity changes on arbitrary threads using [CommandBuffer](https://github.com/friflo/Friflo.Engine-docs/blob/main/api/CommandBuffer.md)'s.
- Build a **hierarchy of entities** typically used in Games and Game Editors.
- Support **multi threaded** component queries (systems).
- Support for **Vectorization (SIMD)** of components returned by queries.  
  Returned component arrays have padding elements at the end to enable SIMD processing without a
  [scalar remainder (epilogue) loop](https://llvm.org/docs/Vectorizers.html#epilogue-vectorization).  
  It is preferred over multi threading as it uses only one core providing the same performance as multi threading running on all cores.
- Minimize times required for GC collection by using struct types for entities and components.  
  GC.Collect(1) < 0.8 ms when using 10.000.000 entities.
- Support **tagging** of entities and use them as a filter in queries.
- Add scripts - similar to `MonoBehavior`'s - to entities in cases OOP is preferred.
- Support **observing entity changes** by event handlers triggered by adding / removing: components, tags, scripts and child entities.
- Reliability - no undefined behavior with only one exception:  
  Performing structural changes - adding/removing components/tags - while iterating a query result.  
  The solution is buffering structural changes with a CommandBuffer.


## Other
- Enable binding an entity hierarchy to a [TreeDataGrid - GitHub](https://github.com/AvaloniaUI/Avalonia.Controls.TreeDataGrid)
  in [AvaloniaUI - Website](https://avaloniaui.net/). Screenshot below:    
<img src="https://raw.githubusercontent.com/friflo/Friflo.Json.Fliox/main/Engine/docs/images/Friflo-Engine-Editor.png" width="677" height="371"></img>

