
**Friflo.Engine.ECS** supports [Native AOT deployment](https://learn.microsoft.com/en-us/dotnet/core/deploying/native-aot).

> *Note*  
> JSON serialization is currently not support by Native AOT.

Using **Friflo.Engine.ECS** does not require a source generator - aka Roslyn Analyzer.  
A source generator could be used to register component types automatically.

Because of this component types used by an application must be registered on startup as shown below.

```cs
var aot = new NativeAOT();
aot.RegisterComponent<MyComponent>();
aot.RegisterTag      <MyTag1>();
aot.RegisterScript   <MyScript>();
var schema = aot.CreateSchema();
```

In case using an unregistered component a `TypeInitializationException` will be thrown. E.g.
```cs
 entity.AddComponent(new UnregisteredComponent());
```

On console to the exception log looks like
```
A type initializer threw an exception. To determine which type, inspect the InnerException's StackTrace property.
Stack Trace:
   at System.Runtime.CompilerServices.ClassConstructorRunner.EnsureClassConstructorRun(StaticClassConstructionContext*) + 0x14f
   at System.Runtime.CompilerServices.ClassConstructorRunner.CheckStaticClassConstructionReturnNonGCStaticBase(StaticClassConstructionContext*, IntPtr) + 0x1c
   at Friflo.Engine.ECS.Entity.AddComponent[T](T&) + 0x4e
   at MyApplication.UseUnregisteredComponent() + 0x7c
```