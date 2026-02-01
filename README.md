![friflo ECS](images/friflo-ECS.svg)


**friflo ECS**, a small and high-performance **C# ECS** - Entity Component System.  
Its an archetype based ECS with focus on performance and simplicity.

<details>
<summary>🛈 What is an Entity Component System?</summary>

An Entity Component System (**ECS**) is a software architecture pattern. [Wikipedia](https://en.wikipedia.org/wiki/Entity_component_system).  
It is often used in software development for **Games**, **Simulation**, **Analytics** and **In-Memory Database** providing high performant data processing.

An ECS has two major strengths:

1. It enables writing **highly decoupled code**. Data is stored in **Components** which are assigned to objects - aka **Entities** - at runtime.  
   Code decoupling is accomplished by dividing implementation in pure data structures (**Component types**) - and code (**Systems**) to process them.  
  
2. It provides **high performant query execution** by storing components in continuous memory to leverage L1 CPU cache and its prefetcher.  
   It improves CPU branch prediction by minimizing conditional branches when processing components in tight loops. 
   [Data-oriented design ⋅ Wikipedia](https://en.wikipedia.org/wiki/Data-oriented_design).
</details>

## Design goals

<details>
<summary>🔥 <strong>High Performance</strong></summary>
Optimal and efficient query / system execution.<br/>
Fast entity creation and component changes.
</details>

<details>
<summary>🎯 <strong>Simple</strong></summary>
Simple API - convenient to debug. No boilerplate code.<br/>
CLS Compliant API - supporting all .NET languages: C#, F#, VB.NET,...

</details>

<details>
<summary>🔄 <strong>Low memory footprint</strong></summary>
Minimal heap allocations at start phase.<br/>
No heap allocations after internal buffers grown large enough.<br/>
No GC pauses / no frame drops.
</details>

<details>
<summary>🛡️ <strong>Reliable</strong></summary>
100% verifiably <strong>safe C#</strong>. No <strong>unsafe</strong> code or native bindings.<br/>
Full test coverage. Expressive runtime errors. Actively maintained.<br/>
Your code requires no <strong>Unsafe</strong> quirks for maximum performance.
</details>

<details>
<summary>🤏 <strong>Small</strong></summary>
Friflo.Engine.ECS.dll size: only 320 kb.  <br/>
No code generation. No 3rd party dependencies.
</details>


# Overview

A common ECS provide the basic features listed bellow.  
To solve other common use-cases not covered by basic implementations this ECS provide the listed extensions.

## Basic features

> An ECS acts like an in-memory database and stores entities in an [EntityStore](docs/entity.md#entitystore) aka *World*.  
> An [Entity](docs/entity.md) is a value type - aka `struct` - with a unique `Id`.

> Data is stored in [Components](docs/entity.md#component) added to entities.  
> Components are stored highly packed in continuous memory.

> The major strength of an archetype based ECS is efficient and performant [Query](docs/query.md) execution.  
> [Query filters](docs/query.md#query-filter) are used to reduce the amount of components returned by the query result.


## Extended features

> Support [Events](docs/events.md) to subscribe entity changes like adding or removing components.  
> Event handlers can be added to a single `Entity` or the entire `EntityStore`.

> [Index / Search](docs/component-index.md) to enable lookup entities with specific component values. **v3.0.0**  
> A lookup for a specific component value - aka search - executes in **O(1)** ~ 4 ns.  
> *Possibly the first and only ECS that supports indexing.*

> [Relationships](docs/relationships.md) to create links between entities. **v3.0.0**  
> A link is directed and bidirectional. Deleting *source* or *target* entity removes a link.

> [Relations](docs/relations.md) to add multiple *"components"* to an entity. **v3.0.0**  
> Relations are not implemented as components to avoid *archetype fragmentation*.

> [Systems](docs/systems.md) are optional. They are used to group queries or custom operations.  
> They support logging / monitoring of execution times and memory allocations in realtime.

> [Hierarchy / Scene tree](docs/entity.md#hierarchy) used to setup a child/parent relationship between entities.  
> An entity in the hierarchy provide direct access to its children and parent.

> [JSON Serialization](docs/entity.md#json-serialization) to serialize entities or the entire EntityStore as JSON.



## Library

> 100% verifiably safe 🔒 C#. No [*unsafe code*](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/unsafe-code)
> or native bindings.  
> This prevents *segmentation faults*, *access violations* and *memory corruption*.

> Minimal heap allocations.  
> After internal buffers grown large enough no heap allocation will occur.  
> At this point no garbage collection will be executed. This avoids frame stuttering / lagging caused by GC runs.  
> Especially no allocation when
> - Creating or deleting entities
> - Adding or removing components / tags
> - Adding or removing relations or relationships
> - Emitting events
> - Changes in entity hierarchy
> - Query or system execution

> Aims for 100% test coverage:  
> Code coverage: [![codecov](https://img.shields.io/codecov/c/gh/friflo/Friflo.Engine.ECS?logoColor=white&label=codecov)](https://app.codecov.io/gh/friflo/Friflo.Engine.ECS/tree/main/src/ECS)

<br>


# Project

[Release Notes](package/Release-Notes.md) - document all nuget releases.

[Library](package/Library.md) describes assembly specific characteristics.

[Unity Extension](extensions/Unity-extension.md) with ECS integration in Unity Editor.


## External Links

GitHub: [https://github.com/friflo/Friflo.Engine.ECS](https://github.com/friflo/Friflo.Engine.ECS)

Discord: [friflo ECS](https://discord.gg/nFfrhgQkb8)

Benchmark: [C# ECS Benchmarks](https://github.com/friflo/ECS.CSharp.Benchmark-common-use-cases)

<br/>


## GitHub

[![Star History Chart](https://api.star-history.com/svg?repos=friflo/Friflo.Engine.ECS)](https://github.com/friflo/Friflo.Engine.ECS)

💖 In case you like this project?  
Leave a ⭐ on [GitHub ⋅ friflo/Friflo.Engine.ECS](https://github.com/friflo/Friflo.Engine.ECS)
