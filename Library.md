
The library can be build on all platforms a .NET SDK is available.  
Build options:
- `dotnet` CLI       - Windows, macOS, Linux
- Rider              - Windows, macOS, Linux (untested)
- Visual Studio 2022 - Windows
- Visual Studio Code - Windows, macOS, Linux (untested)


## Supported Platforms
- Builds tested on:  
  **Windows, macOS, Linux, WASM / WebAssembly, Unity, Godot, MonoGame**.  


## Assembly (dll)
- This library is using only *verifiably safe code*. `<AllowUnsafeBlocks>false</AllowUnsafeBlocks>`.  
  No use of [Unsafe code ⋅ Microsoft](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/unsafe-code) -
  *"Unsafe code may cause memory corruption and introduces security and stability risks"*.

  Access violation bugs caused by unsafe code on customer installs provide often no stacktrace when crashing.

  Typical random behavior using **unsafe code**:
  - Expected behavior.
  - An `AccessViolationException` while debugging. Best case. The stacktrace on dev machine may show the root cause.  
  - A [Segmentation fault ⋅ Wikipedia](https://en.wikipedia.org/wiki/Segmentation_fault) on customer installs without stacktrace.
  - An `Exception` which is unrelated to the root cause.
  - native memory: leaks, dangling pointers or wild pointers
  - Nothing happens - no exception. Worst case.
  - Exploit execution of malicious code. Even worse. 😁

  Behavior of **safe code**:
  - Expected behavior.
  - An `Exception` showing the root cause of a bug.

- Pure C# implementation - no C/C++ bindings slowing down runtime / development performance.

- The C# API is [CLS-compliant ⋅ Microsoft](https://learn.microsoft.com/en-us/dotnet/api/system.clscompliantattribute?view=net-8.0#remarks).

- No custom C# preprocessor directives which requires custom builds to enable / disable features.

- Deterministic dll build.

- No 3rd party dependencies.

- It requires **Friflo.Json.Fliox** which is part of this repository.


## Build
- Size: Friflo.Engine.ECS.dll: ~ 210 kb. Implementation: ~ 17.500 LOC.

- Build time Windows: ~ 5 seconds, macOS (M2): 2,5 seconds.

- Code coverage of the unit tests: 99,9%. See: [code-coverage.md](https://github.com/friflo/Friflo.Json.Fliox/blob/main/Engine/docs/code-coverage.md).

- Unit test execution: ~ 1 second.

- The nuget package contains four dll's specific for: **.NET Standard 2.1 .NET 6 .NET 7 .NET 8**.  
  This enables using the most performant features available for each target.  
  E.g. Some SIMD intrinsics methods available on .NET 7 and .NET 8 but not on earlier versions.
