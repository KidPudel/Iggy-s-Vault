**Resources are data containers designed for serialization.** They inherit from `RefCounted`, meaning they're automatically freed when no longer referenced.
**Resources are for data you need to reference/modify/save independently of where it's displayed.**

## Core Concept

By extending a class with `Resource`, you gain the ability to export and persist data. Resources model **static data** — definitions, templates, configurations.

## Serialization Behavior

When Godot saves/loads a Resource (`.tres` file), it:

1. Creates a blank instance (`_init()` with no arguments)
2. Populates `@export` properties from the file

Because godot needs to figure out which properties to write to the `.tres` file. It does this by:

1. Creating a fresh "default" instance (no args)
2. Comparing your actual instance against this default
3. Only saving properties that differ from defaults

This is an optimization — it avoids writing default values to disk, keeping save files smaller.

!!! Constructor arguments are not stored — only exported properties. This means `_init()` must be able to run without arguments, with actual setup happening after properties are assigned.

**The idiomatic solution:** Either guard against null in `_init()`, or move `_prepare_states()` to be called lazily when `state_textures` is actually needed.


## Loading Resources

```gdscript
# Compile-time loading (cached immediately)
var scene = preload("res://player.tscn")

# Runtime loading (cached on first load)
var scene = load("res://player.tscn")

# Threaded loading (for large resources)
ResourceLoader.load_threaded_request("res://large_scene.tscn")
# ... later ...
var scene = ResourceLoader.load_threaded_get("res://large_scene.tscn")
```

Godot maintains a global cache by path. First load hits disk; subsequent loads return the cached instance until all references are released.

Use `ResourceLoader.CACHE_MODE_IGNORE` when you need independent copies.

## Common Resource Types

- `PackedScene` — stores and instantiates scenes
- Textures, audio, animations
- Collision shapes
- Custom data resources

## Data Architecture Guidelines

**Ownership flows downward:** Store "owns many" relationships on the parent (e.g., `Inventory` holds an array of `Items`), not "belongs to" on children. This aligns with Godot's hierarchical design, simplifies serialization, and avoids dangling references.

**Separate static from mutable:**

- `res://` — immutable definitions, templates (design-time data)
- `user://` — save data, runtime state (mutable data)