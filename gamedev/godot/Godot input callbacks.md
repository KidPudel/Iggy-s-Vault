# Godot input callbacks

## Definition

Godot exposes callback methods that receive `InputEvent` objects directly.

## Core callbacks

- `_unhandled_input(event)`: gameplay controls after UI.
- `_input(event)`: global early input.
- `_gui_input(event)`: Control nodes only.
- `_shortcut_input(event)`: shortcut-focused stage.
- `_unhandled_key_input(event)`: keyboard-only unhandled stage.

## Practical mapping

- Gameplay: `_unhandled_input()`
- Global/system hotkeys: `_input()` or `_shortcut_input()`
- UI widgets: `_gui_input()`

## Why it matters

Callback choice determines whether UI blocks gameplay and where conflicts occur.

## See also

- [[Godot input propagation]]
- [[Input singleton in Godot]]
- [[input handling reference in Godot]]
