[![friflo ECS](images/friflo-ECS.svg)](https://github.com/friflo/Friflo.Engine.ECS)   ![splash](images/paint-splatter.svg)

[![Github Repo](images/github-mark.svg)](https://github.com/friflo/Friflo.Engine.ECS)
[![Github Repo](https://img.shields.io/badge/Repository-grey)](https://github.com/friflo/Friflo.Engine.ECS)
[![Demos](https://img.shields.io/badge/Demos-22aa22)](https://github.com/friflo/Friflo.Engine.ECS-Demos)
[![C# API](https://img.shields.io/badge/C%23%20API-22aaaa)](https://github.com/friflo/Friflo.Engine-docs)
[![Discord](https://img.shields.io/badge/Discord-5865F2)](https://discord.gg/nFfrhgQkb8)   
[![Benchmarks](https://img.shields.io/badge/Benchmark%20🏁%20of%20C%23%20ECS%20frameworks-ffffff)](https://github.com/friflo/ECS.CSharp.Benchmark-common-use-cases)



**friflo ECS** is an archetype based ECS - Entity Component System.  
It set its focus on high-performance, low-memory footprint and minimal heap allocations.

<details>

<summary>What is an Entity Component System?</summary>

An Entity Component System (**ECS**) is a software architecture pattern. See [ECS ⋅ Wikipedia](https://en.wikipedia.org/wiki/Entity_component_system).  
It is often used in software development for **Games**, **Simulation**, **Analytics** and **In-Memory Database** providing high performant data processing.

An ECS has two major strengths:

1. It enables writing **highly decoupled code**. Data is stored in **Components** which are assigned to objects - aka **Entities** - at runtime.  
   Code decoupling is accomplished by dividing implementation in pure data structures (**Component types**) - and code (**Systems**) to process them.  
  
2. It provides **high performant query execution** by storing components in continuous memory to leverage L1 CPU cache and its prefetcher.  
   It improves CPU branch prediction by minimizing conditional branches when processing components in tight loops.
   See [DoD - Data-oriented design](https://en.wikipedia.org/wiki/Data-oriented_design).

</details>


# Overview

A common ECS provide the basic features listed bellow.  
To solve other common use-cases non covered by a basic implementation this ECS provide the listed extensions.

## Basic features

> An ECS acts like an in-memory database and stores entities in an [EntityStore](docs/entity.md#entitystore).  
> An [Entity](docs/entity.md) is a value type - aka `struct` - with a unique `Id`.

> Data is stored in [Components](docs/entity.md#component) added to entities.  
> Components are stored highly packed in continuous memory.

> The major strength of an archetype based ECS is efficient and performant [Query](docs/query.md) execution.  
> [Query filters](docs/query.md#query-filter) are used to reduce the amount of components returned by the query result.


## Extended features

> Support [Events](docs/events.md) to subscribe entity changes like adding or removing components.  
> Event handlers can be added to a single `Entity` or the entire `EntityStore`.

> [Index / Search](docs/component-index.md) to enable lookup entities with specific component values.  
> A lookup for a specific component value - aka search - executes in **O(1)** ~ 4 ns.  
> *Possibly the first and only ECS that supports indexing.*

> [Relationships](docs/relationships.md) to create links between entities.  
> Links are directed and bidirectional.

> [Relations](docs/relations.md) to add multiple *"components"* to an entity.  
> Relations are not implemented as components to avoid *archetype fragmentation*.

> [Hierarchy / Scene tree](docs/entity.md#hierarchy) used to setup a child/parent relationship between entities.  
> An entity in the hierarchy provide direct access to its children and parent.

> [Systems](docs/systems.md) are optional. They are used to group queries or custom operations.  
> Systems support logging and realtime monitoring to find bottlenecks.


## Library features

> 100% verifiably safe 🔒 C#. No [*unsafe code*](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/unsafe-code).  
> This avoids surprises getting *segmentation faults* or *access violations* leading to instant process termination.

> Aims for 100% test coverage: See [![codecov](https://img.shields.io/codecov/c/gh/friflo/Friflo.Engine.ECS?logo=codecov&logoColor=white&label=codecov)](https://app.codecov.io/gh/friflo/Friflo.Engine.ECS/tree/main/src/ECS)

<br>


# Project

[Release Notes](package/Release-Notes.md) to document all nuget releases.

[Library](package/Library.md) describes assembly specific characteristics.

[Unity Extension](extensions/Unity-extension.md) with ECS integration in Unity Editor.


## External Links

GitHub: [Friflo.Engine.ECS](https://github.com/friflo/Friflo.Engine.ECS)

Discord: [friflo ECS](https://discord.gg/nFfrhgQkb8)

Benchmark: [C# ECS Benchmarks](https://github.com/friflo/ECS.CSharp.Benchmark-common-use-cases)

<br/>


## GitHub

[![Star History Chart](https://api.star-history.com/svg?repos=friflo/Friflo.Engine.ECS&type=Timeline)](https://github.com/friflo/Friflo.Engine.ECS)

💖 In case you like this project?  
Leave a ⭐ on [GitHub ⋅ friflo/Friflo.Engine.ECS](https://github.com/friflo/Friflo.Engine.ECS)
