Translated with [Claude AI](https://claude.ai).


# LeoECS Lite Additional Systems.
Additional systems extending the functionality of LeoECS Lite.

> Tested on Unity 2020.3 (does not depend on Unity) and contains asmdef descriptions for compilation as separate assemblies and reducing main project recompilation time.

# Contents
* [Social links](#social-links)  
* [Installation](#installation)
  * [As Unity module](#as-unity-module)
  * [As source code](#as-source-code)
* [Classes](#classes)
  * [EcsGroupSystem](#ecsgroupsystem)
  * [DelHere](#delhere)
* [License](#license)
* [FAQ](#faq)

# Social links
[![discord](https://img.shields.io/discord/404358247621853185.svg?label=enter%20discord%20server&style=for-the-badge&logo=discord)](https://discord.gg/5GZVde6)

# Installation

## As Unity module 
Installation as Unity module via git url in PackageManager or direct editing of `Packages/manifest.json` is supported:
```
"com.leopotam.ecslite.extendedsystems": "https://github.com/Leopotam/ecslite-extendedsystems.git",
```
By default the latest release version is used. If you need the "in development" version with the latest changes - you need to switch to the `develop` branch:
```
"com.leopotam.ecslite.extendedsystems": "https://github.com/Leopotam/ecslite-extendedsystems.git#develop",
```

## As source code
The code can also be cloned or downloaded as an archive from the releases page.

# Classes

## EcsGroupSystem
`EcsGroupSystem` allows grouping ECS systems into groups with the ability to enable/disable them during runtime:
```c# 
// These systems will be nested in a group named "Melee".
class MeleeSystem1 : IEcsRunSystem {
  public void Run (IEcsSystems systems) { }
}
class MeleeSystem2 : IEcsRunSystem {
  public void Run (IEcsSystems systems) { }  
}

class MeleeGroupEnableSystem : IEcsRunSystem {
  public void Run (IEcsSystems systems) {
    // We can enable and disable the "Melee" group by 
    // sending the special "EcsGroupSystemState" event.  
    var world = systems.GetWorld ();
    var entity = world.NewEntity ();
    ref var evt = ref world.GetPool<EcsGroupSystemState> ().Add (entity);
    evt.Name = "Melee";
    evt.State = true;
  }
}
...
// Initialization.
var systems = new EcsSystems (new EcsWorld ());
systems
  // Add the "Melee" group disabled initially with 2 nested
  // systems, group state change events will be stored 
  // in the default world.
  .AddGroup ("Melee", false, null, 
    new MeleeSystem1 (),
    new MeleeSystem2 ())
  // Other systems.  
  .Add (new MeleeGroupEnableSystem ())
  .Init ();
```

## DelHere
`DelHere` is a helper behavior to automatically clean up event components at the specified location:
```c#
var systems = new EcsSystems (new EcsWorld ());
systems
  .Add (new System1 ()) 
  .Add (new System2 ())
  // All "C1" components will be removed here.
  .DelHere<C1> () 
  .Add (new System3 ())
  // All "C2" components will be removed here.
  .DelHere<C2> ()
  .Add (new System4 ())
  .Init ();
```
> **IMPORTANT!** If `DelHere` deletes components from explicitly specified world - this world should be registered via `AddWorld()` call before `DelHere()`.

# License
The framework is released under two licenses, see [details here](./LICENSE.md).

In cases of licensing under MIT-Red terms, you should not expect any personal support or guarantees of any kind.

# FAQ

### I want to use the dependency injection functionality from `ecslite-di`, but the fields of systems nested in named groups are not initialized properly. How can I fix this?

It is enough to specify the `LEOECSLITE_DI` compiler directive.
