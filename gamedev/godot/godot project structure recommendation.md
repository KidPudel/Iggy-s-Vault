Treat **each feature (scene, character, UI screen, level, etc.) as its own little island**: keep its .tscn, script, textures, sounds and other resources in the _same_ (outer) folder, and reserve only a handful of top-level folders for truly global things like autoloads or third-party add-ons. Godot’s own docs – recommend this in game functionality based approach because it scales far better than sorting by file type.

Just like in any project, separate projects by logical components you build and sort them as they grow (or as you know they will grow).

> **REMEMBER**: it is all just recommendations and general guidelines, not the strict rules (like most of the time), just read it, take into account and do what you want. Free will. ***AS ALWAYS, IT DEPENDS.***

## **A practical top-level layout**

We try to group them by the most common delimiters/categories as possible, and logically by the dependency.
```
res://
  project.godot
  addons/          # third-party plugins or assets
  common/          # standalone
  utilities/       # helpers like game and level manager, 
  ui/              # menus & HUD that are reused everywhere
  actors/          # player, enemies, NPCs – one sub-folder each
  levels/          # each playable level/world map scene
  assets_raw/      # .blend, .psd, wav masters – ignored by Godot
  assets/          # global to the game, like soundtrack.
```

```
actors/
  player/
    player.tscn
    player.gd
    art/
      run.png  walk.png
    sounds/
      jump.wav  hurt.wav
	data/
	  resource_file.tscn
  goblin/
    goblin.tscn
    goblin.gd
    goblin.png
```

## **Naming & style conventions that save headaches**
- **snake_case** all folders and filenames (player_attack.gd), except C# scripts which follow the C# PascalCase class name.
- **PascalCase** node names inside the scene tree (Player, MainMenu).
- Keep _all_ third-party items in addons/ so you can instantly see what’s yours and what isn’t. 
- Drop an **empty .gdignore** file in assets_raw/ (or any folder you don’t want imported) so Godot skips it during import. 

Placing an **empty file named .gdignore inside a folder tells Godot to skip that folder entirely in step 1 above**. The editor won’t:
- Parse or display any of the files in the FileSystem dock.
- Generate .import metadata.
- Re-import when those files change.


# Main scene
>   General good practice:  
>   A top level “Main” basic node, with two children: A “UI” control node, and a “World” Node2D/3D
>   Menus/HUDs spawn under UI, and levels/maps spawn under World
>   You can have an autoload, or the Main node, or the UI/World nodes handle the loading/spawning/switching
>   
> **NOTE:** It all depends on what you want to do.

Main scene is just an *initial* entry point of our game. we build on that base.
Note it is not the root, the root scene is always present in a tree and it is type of [[viewport in godot]]
```gdscript
get_tree().root # get root
get_tree().current_scene # get active scene under the root
```

## What it Should Not Be
- A monolithic script-heavy manager with all logic
- Constantly replaced (e.g., via change_scene_to)
- Responsible for too much – offload to autoloads or scene scripts

## Additional Good Practice
- Scene transitions: swap under World node, not by replacing Main
- Use autoloads for truly global concerns only (settings, save, game state, etc.)
- Treat each level, actor, and UI screen as its own self-contained folder and scene


```
Main (Node2D)
├── World (Node2D)         # Where levels are added dynamically
├── UI (CanvasLayer)       # Pause menu, HUD, etc.
```


## Mental model for semantics of base components

Why build mental model

Yes, everything is data in computer science, nothing less, nothing more, but since I work with a Godot engine, an existing engine, and developers designed the tools and components the specific way, so you must, as the user, use the engine the intended way, and for that you need to build a mental model to help you, not just data, because, yes, some stuff like autoloads, for example, singleton, you can use in 100,000 ways, but without a mental model you can pollute your code, make it unbearable, and stuff like that, and much much more, and a lot of bad practices can be slipped into the project, so this is why the mental model is important.

We can imagine game as a theater play.


Engine is a global runtime. theatre, the whole physical thing where everthing is happening


Root node is the viewport. The physical stage, performance space, it is completely static. It provides a way to put the play in it.
Root node is the engine tree entry, to which your game is hooked.


Main scene is a current active stage setup it has its own global and *fundamentally unique model/purpose of existence*. So if we want to represent something entirely new and its own, then it is a good practice to replace the stage itself.
Example would be the main menu, and the main game, credits.
Subtree node that is the holder of the current game content


The nested content in the main scene is like an actors and props.