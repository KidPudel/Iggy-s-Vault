# Godot @tool Reference

## What It Does

`@tool` makes your script run in the editor, not just at runtime. Without it, your script code only executes when the game is playing.

```gdscript
@tool
extends Node2D
```

## When You Need It

| Situation                                 | @tool needed?  |
| ----------------------------------------- | -------------- |
| Shader preview in editor                  | No (automatic) |
| Script changes node properties in editor  | Yes            |
| @export variable triggers code via setter | Yes            |
| Visual preview of procedural generation   | Yes            |
| Custom gizmos/handles                     | Yes            |
| Just using @export for data               | No             |

## Common Pattern: Export with Setter

```
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

## Gotchas

### 1. _ready() runs in editor

```gdscript
@tool
extends Node2D

func _ready():
    # This runs when scene opens in editor AND at runtime
    if Engine.is_editor_hint():
        return  # Skip editor-only
    
    # Runtime-only code here
```

### 2. _process() runs in editor

```gdscript
@tool
extends Node2D

func _process(delta):
    if Engine.is_editor_hint():
        # Editor-only behavior
        queue_redraw()
    else:
        # Game behavior
        move_and_slide()
```

### 3. Accidental scene modifications

Tool scripts can modify your scene and trigger saves. Be careful with code that adds/removes nodes or changes properties unexpectedly.

## Engine.is_editor_hint()

Your best friend with `@tool`. Returns `true` when running in editor, `false` at runtime.

```gdscript
@tool
extends CharacterBody2D

func _physics_process(delta):
    if Engine.is_editor_hint():
        return  # Don't run physics in editor
    
    velocity.y += gravity * delta
    move_and_slide()
```

## Performance

`@tool` has **zero runtime performance impact**. The game doesn't care whether the annotation exists—it runs identically either way.

## Quick Checklist

Before adding `@tool`, ask:

1. Does my script need to **execute code** in the editor? (not just expose variables)
2. Am I prepared to guard runtime-only code with `is_editor_hint()`?
3. Will this accidentally modify my scene?

If just exposing data via `@export` with no logic, you don't need `@tool`.