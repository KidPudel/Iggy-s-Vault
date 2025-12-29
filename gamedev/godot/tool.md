# @tool Annotation

## What It Does

`@tool` makes your script run in the editor, not just at runtime. Without it, your script code only executes when the game is playing.

```gdscript
@tool
extends Node2D
```

## When You Need It

|Situation|@tool needed?|
|---|---|
|Shader preview in editor|No (automatic)|
|Script changes node properties in editor|Yes|
|@export variable triggers code via setter|Yes|
|Visual preview of procedural generation|Yes|
|Custom gizmos/handles|Yes|
|Just using @export for data|No|

## Common Pattern: Export with Setter

```gdscript
@tool
extends MeshInstance2D

@export var size: Vector2 = Vector2(100, 50):
    set(value):
        size = value
        _update_mesh()

func _update_mesh():
    if mesh:
        mesh.size = size
```

Without `@tool`, the setter runs only at runtime. With `@tool`, it runs in the editor too.

## Button-Like Export Pattern

Use a bool export as a clickable action trigger in the inspector:

```gdscript
@tool
extends Node3D

@export var generate: bool = false:
    set(value):
        if value:
            _do_generate()
        generate = false  # Always reset so it can be clicked again

func _do_generate():
    if not Engine.is_editor_hint():
        return
    print("Generating...")
    # Your logic here
```

The `if value:` check prevents the function from firing when resetting to false.

## Lifecycle Methods in Editor

### _ready() runs in editor

```gdscript
@tool
extends Node2D

func _ready():
    # This runs when scene opens in editor AND at runtime
    if Engine.is_editor_hint():
        return  # Skip in editor
    
    # Runtime-only code here
```

### _enter_tree() is more reliable

For editor setup, `_enter_tree()` is often more reliable than `_ready()`:

```gdscript
@tool
extends Node3D

func _enter_tree():
    if Engine.is_editor_hint():
        _ensure_children.call_deferred()

func _ensure_children():
    if not has_node("Markers"):
        var markers = Node3D.new()
        markers.name = "Markers"
        add_child(markers)
        markers.owner = get_tree().edited_scene_root
```

### _process() runs in editor

```gdscript
@tool
extends Node2D

func _process(delta):
    if Engine.is_editor_hint():
        queue_redraw()  # Editor-only behavior
    else:
        move_and_slide()  # Game behavior
```

### _input() does NOT work in editor

`_input()` is not called for `@tool` scripts in the editor. The editor consumes input events before they reach your script. For editor input handling, you need an EditorPlugin (see below).

## Adding Children Programmatically

When adding children in the editor, you must set `owner` for them to save with the scene:

```gdscript
@tool
extends Node3D

func _enter_tree():
    if Engine.is_editor_hint():
        _setup_children.call_deferred()

func _setup_children():
    if not has_node("Generated"):
        var node = Node3D.new()
        node.name = "Generated"
        add_child(node)
        node.owner = get_tree().edited_scene_root  # Critical for saving
```

The `owner` must be the scene root (not the parent node) because Godot only serializes nodes whose owner matches the scene being saved.

## Engine.is_editor_hint()

Returns `true` when running in editor, `false` at runtime.

```gdscript
@tool
extends CharacterBody2D

func _physics_process(delta):
    if Engine.is_editor_hint():
        return  # Don't run physics in editor
    
    velocity.y += gravity * delta
    move_and_slide()
```

## Gotchas

1. **Accidental scene modifications**: Tool scripts can modify your scene and trigger saves. Be careful with code that adds/removes nodes or changes properties unexpectedly.
    
2. **Script changes require reload**: After editing `@tool` scripts, Godot doesn't always pick up changes. Use Ctrl+Shift+X to reload the project, or remove and re-add the node.
    
3. **print() outputs to Output panel**: Use `print()` for debugging—it shows in the editor's Output panel at the bottom.
    

## Performance

`@tool` has **zero runtime performance impact**. The game doesn't care whether the annotation exists—it runs identically either way.

## Quick Checklist

Before adding `@tool`, ask:

1. Does my script need to **execute code** in the editor? (not just expose variables)
2. Am I prepared to guard runtime-only code with `is_editor_hint()`?
3. Will this accidentally modify my scene?

If just exposing data via `@export` with no logic, you don't need `@tool`.


---

# @tool Essentials

```gdscript
@tool                              # First line of script
Engine.is_editor_hint()            # Check if running in editor
get_tree().edited_scene_root       # Root of scene being edited
node.owner = edited_scene_root     # Required for saving children
```
