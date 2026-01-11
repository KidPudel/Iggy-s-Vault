
# Godot Resource

> [!quote] Core Principle Resources are data containers designed for serialization that exist outside the scene tree.

---

## What is a Resource?

**In one sentence:** A Resource is a data object you can save to a file, share between nodes, and serialize automatically.

Resources are Godot's solution for:

- Storing data independently of nodes
- Sharing data between multiple nodes
- Serializing data to disk (`.tres` files)
- Creating reusable templates and configurations

Resources inherit from `RefCounted`, meaning they're automatically freed when no longer referenced—no manual memory management needed.

---

## When to Use Resources

**Use Resources when you need data that:**

- Lives independently of any specific node
- Can be shared between multiple nodes
- Needs to be saved/loaded from disk
- Represents configuration, definitions, or templates

**Examples:**

- Item definitions (stats, icons, descriptions)
- Character stats (health, damage, speed)
- Game configuration (settings, difficulty presets)
- Save game data (player progress, inventory)

**Don't use Resources for:**

- Data tightly coupled to a single node's lifetime (use node properties)
- Temporary runtime calculations (use variables)
- Data that never needs serialization (use classes)

---

## Core Concept

By extending `Resource`, you gain automatic serialization:

```gdscript
class_name ItemData
extends Resource

@export var id: StringName
@export var display_name: String
@export var damage: int = 10
@export var icon: Texture2D
```

Resources model **static data** — definitions, templates, and configurations. They're read-only blueprints that define "what something can be."

---

## How Serialization Works

Understanding Godot's serialization process is critical to avoiding bugs.

### The Save/Load Process

When Godot saves a Resource to a `.tres` file:

1. **Creates a default instance** — Calls `_init()` with no arguments
2. **Compares against your actual instance** — Identifies which properties differ from defaults
3. **Saves only the differences** — Optimizes file size by skipping default values

When Godot loads a Resource from a `.tres` file:

1. **Creates a blank instance** — Calls `_init()` with no arguments
2. **Populates `@export` properties** — Reads values from the file
3. **Returns the instance** — Ready to use

### Critical Implication

> [!danger] Constructor Arguments Are Not Stored Only `@export` properties are serialized. Constructor parameters are lost on save/load.

This means:

- `_init()` **must work with no arguments**
- Actual initialization should happen **after** properties are loaded
- Any computed values should be created **lazily** or in a separate setup method

### Example: The Right Way

```gdscript
class_name ItemData
extends Resource

@export var id: StringName
@export var damage: int = 10

# Lazy initialization - computed only when needed
var _computed_stats: Dictionary

func get_stats() -> Dictionary:
    if not _computed_stats:
        _compute_stats()
    return _computed_stats

func _compute_stats() -> void:
    _computed_stats = {
        "dps": damage * 1.5,
        "crit": damage * 2.0
    }
```

### Example: The Wrong Way

```gdscript
class_name ItemData
extends Resource

@export var id: StringName
@export var damage: int = 10

var computed_stats: Dictionary

func _init(item_id: StringName = "") -> void:
    id = item_id
    # ❌ This breaks on load - id isn't set yet!
    computed_stats = _compute_stats()
```

**Why this breaks:**

1. On load, Godot calls `_init()` with no arguments
2. `id` is empty at this point
3. `_compute_stats()` runs with invalid data
4. Later, Godot sets `id` from the file, but it's too late

---

## Loading Resources

```gdscript
# Compile-time loading (cached immediately at startup)
var data = preload("res://data/items/sword.tres")

# Runtime loading (cached on first load)
var data = load("res://data/items/sword.tres")

# Threaded loading (for large resources, prevents frame drops)
ResourceLoader.load_threaded_request("res://scenes/large_level.tscn")
# ... do other work ...
var scene = ResourceLoader.load_threaded_get("res://scenes/large_level.tscn")
```

### Resource Caching

Godot maintains a **global cache by file path**:

- First `load()` reads from disk and caches the result
- Subsequent `load()` calls return the **same cached instance**
- Cache persists until all references are released

**Implications:**

- Multiple nodes loading the same resource get the **same object**
- Modifying a loaded resource affects **all nodes using it**
- This is intentional for read-only data (templates, definitions)

### When You Need Independent Copies

Use `CACHE_MODE_IGNORE` to force a fresh copy:

```gdscript
# Each load creates a new, independent instance
var save = ResourceLoader.load(
    "user://saves/slot_1.tres",
    "",
    ResourceLoader.CACHE_MODE_IGNORE
)
```

> [!warning] Always Use CACHE_MODE_IGNORE for Save Files Without it, loading the same save file twice returns the cached instance. Modifying it corrupts the cache, breaking subsequent loads.

---

## Common Resource Types

Godot includes many built-in Resource types:

|Type|Purpose|
|---|---|
|`PackedScene`|Stores and instantiates scene trees|
|`Texture2D`|Images, sprites|
|`AudioStream`|Sound effects, music|
|`Animation`|Keyframe animations|
|`Shape3D`|Collision shapes|
|`Material`|Rendering properties|
|`Script`|GDScript, C# code|

You can also create custom Resources for your game's data.

---

## Data Architecture Guidelines

### Ownership Flows Downward

**Store "owns many" relationships on the parent:**

```gdscript
# ✅ Good: Parent owns children
class_name Inventory
extends Resource

@export var items: Array[ItemState] = []

# ❌ Bad: Child points to parent
class_name ItemState
extends Resource

@export var inventory: Inventory  # Creates circular reference
```

**Why:**

- Aligns with Godot's hierarchical design
- Simplifies serialization (no circular references)
- Avoids dangling references when objects are destroyed
- Makes data flow clear and predictable

### Separate Static from Mutable

**Use the right storage location for each data type:**

|Path|Mutability|Use For|
|---|---|---|
|`res://`|Read-only|Definitions, templates, design-time data|
|`user://`|Read-write|Save files, runtime state, settings|

```gdscript
# Static data (shipped with game)
var item_data: ItemData = load("res://data/items/sword.tres")

# Mutable data (player-specific)
ResourceSaver.save(player_state, "user://saves/slot_1.tres")
```

**Key principle:** If data changes during gameplay, it belongs in `user://`. If it's a fixed definition, it belongs in `res://`.

---

## Best Practices

### Constructor Design

```gdscript
class_name MyResource
extends Resource

@export var value: int = 0

# ✅ Always provide defaults for ALL parameters
func _init(v: int = 0) -> void:
    value = v
```

### Lazy Initialization

```gdscript
class_name ItemData
extends Resource

@export var damage: int = 10

var _cached_tooltip: String

func get_tooltip() -> String:
    if not _cached_tooltip:
        _cached_tooltip = "Damage: %d" % damage
    return _cached_tooltip
```

### Avoid Side Effects in _init()

```gdscript
# ❌ Bad: Depends on exported properties
func _init() -> void:
    compute_stats()  # id might not be set yet!

# ✅ Good: Compute on demand
func get_stats() -> Dictionary:
    if not _cached_stats:
        _compute_stats()
    return _cached_stats
```

---

## Quick Reference

### Checklist for Custom Resources

- [ ] Extend `Resource`
- [ ] Mark all saved fields with `@export`
- [ ] Constructor has defaults for all parameters
- [ ] No side effects in `_init()` that depend on exported properties
- [ ] Computed values created lazily, not in constructor
- [ ] Saved to `res://` if static, `user://` if mutable

### Common Issues

|Symptom|Cause|Fix|
|---|---|---|
|"Cannot call _init with 0 arguments"|Constructor requires parameters|Add defaults to all parameters|
|Computed values are wrong on load|Calculated in `_init()` before properties set|Use lazy initialization|
|Multiple loads affect each other|Using cached instance|Use `CACHE_MODE_IGNORE`|
|Circular reference warnings|Child points to parent|Store relationship on parent only|
|Properties not saving|Missing `@export`|Add `@export` to all serialized fields|

---

## Summary

Resources are Godot's data serialization system:

- Exist outside the scene tree
- Automatically saved/loaded with `@export` properties
- Cached by path for memory efficiency
- Perfect for definitions, templates, and save data

**The golden rule:** If data needs to outlive a node or be shared between nodes, use a Resource.

---

_For implementing game data with Resources, see [[Data-Driven Design in Godot]]._