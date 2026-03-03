# Input singleton in Godot

## Definition

The `Input` singleton is Godot's polling API for reading current input state each frame.

## Core methods

- `Input.is_action_pressed("action")`
- `Input.is_action_just_pressed("action")`
- `Input.is_action_just_released("action")`
- `Input.get_vector("left", "right", "up", "down")`
- `Input.get_axis("negative", "positive")`
- `Input.get_action_strength("action")`

## Practical rule

- Use polling in `_process()` or `_physics_process()` for continuous controls (movement, aiming).
- Use callbacks when you need event-specific behavior.

## Why it matters

Separates continuous input logic from event routing logic.

## See also

- [[Godot input callbacks]]
- [[Godot input propagation]]
- [[input handling reference in Godot]]
