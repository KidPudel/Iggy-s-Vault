# Quick Verification

**preload("path")**

- Loads at parse time (when the script file is first parsed - which happens when the scene containing it loads)
- Must be a constant string literal - cannot use variables.
- Same for all instances using that script (baked into the script). It is like [[static]]
- Resource is loaded once and cached - reusing returns the same copy

**@export var x: PackedScene**

- Resources referenced by @export variables are loaded when the scene loads. The same as preload
- Value stored in the .tscn scene file, not the script
- Set per-instance in the inspector - can be different for each instance. This is a crucial difference between preload in use case. It is not static, but is not, it is per-instance.
- Can be changed in editor without modifying code

**ResourceLoader.load("path")** (or just `load()`)

- Loads at runtime when the function is called
- Path can be a variable or expression - supports dynamic paths
- Also uses caching - same resource loaded multiple times returns cached copy
- Most flexible for runtime decisions

# Simple Mental Model

- **preload** = "This script always needs this resource". (static/class-level)
- **@export** = "Configure what resource this instance uses". (per-instance)
- **ResourceLoader.load** = "Figure out and load resource at runtime" (dynamic)

---

# Scenario: Level 2 Scene Script

```gdscript
# level_2.gd (attached to level 2 scene)
extends Node

const PRELOADED_SCENE = preload("res://enemy.tscn")
@export var exported_scene: PackedScene  # Set to boss.tscn in editor
```

## Timeline

**At Game Startup:**

- Level 2 scene is NOT loaded yet
- `level_2.gd` is NOT parsed yet
- `PRELOADED_SCENE` does NOT load yet
- `exported_scene` does NOT load yet

**When You Beat Boss in Level 1:** You call something like:

```gdscript
var level_2 = load("res://level_2.tscn")  # or ResourceLoader.load()
add_child(level_2.instantiate())
```

**What happens in order:**

1. **Level 2 scene file loads** - Godot reads the .tscn file
2. **level_2.gd script is parsed NOW**
    - At this moment, `preload("res://enemy.tscn")` executes
    - enemy.tscn loads into memory immediately
3. **Scene references are loaded**
    - The .tscn file says "exported_scene = boss.tscn"
    - boss.tscn loads into memory now
4. **Scene is instantiated and added to tree**

## Key Point

Both resources load at the same time - when level_2.tscn is loaded. The difference isn't _when_ they load, but:

- **preload** is in the script code (same for all instances)
- **@export** is in the scene file (can differ per instance/scene)

If you had two different versions of level_2.tscn (say, level_2_easy.tscn and level_2_hard.tscn) both using the same script, they could have different values for `exported_scene`, but would both have the same `PRELOADED_SCENE`.


# Scene manager

For level managers specifically, **use paths (hardcoded or @export_file) + ResourceLoader.load()**.

## Why This Is Better

**Memory control:**

```gdscript
# BAD - All 50 levels load into memory immediately
@export var level_1: PackedScene
@export var level_2: PackedScene
# ... 48 more

# GOOD - Levels load only when needed
const LEVELS = {
    1: "res://levels/level_1.tscn",
    2: "res://levels/level_2.tscn",
}
@export_file var path: String

func load_level(id: int):
    var scene = ResourceLoader.load(LEVELS[id])
    return scene.instantiate()
```

**Flexibility:**

```gdscript
# Can load from config files, save data, dynamic strings
var level_path = save_data.get("current_level_path")
StageManager.swap_stage(level_path)
```

**One source of truth:** All level loading goes through your manager - easy to add loading screens, transitions, cleanup, etc.

## The Pattern

```gdscript
# stage_manager.gd
class_name StageManager

func swap_stage(path: String):
    # Clean up current stage
    # Show loading screen
    var new_stage = ResourceLoader.load(path).instantiate()
    # Add to tree
    # Hide loading screen
```

Then everywhere in your code:

```gdscript
StageManager.swap_stage("res://levels/level_5.tscn")
# or
StageManager.swap_stage(LEVELS[current_level_id])
```
