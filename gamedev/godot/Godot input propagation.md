# Godot input propagation

## Definition

Input events flow through multiple stages in order. Earlier handlers can consume the event and stop later handlers.

## Typical flow

1. `_input()`
2. `Control._gui_input()`
3. `_shortcut_input()`
4. `_unhandled_key_input()`
5. `_unhandled_input()`
6. Physics picking

## Handling rule

- For non-UI gameplay input, prefer `_unhandled_input()`.
- Mark event handled when appropriate: `get_viewport().set_input_as_handled()`.

## Why it matters

Understanding propagation prevents conflicts between UI and gameplay controls.

## See also

- [[Godot input callbacks]]
- [[Input singleton in Godot]]
- [[input handling reference in Godot]]
