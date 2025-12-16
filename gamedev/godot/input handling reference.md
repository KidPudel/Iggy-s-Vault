# Godot 4.x Input Handling Reference

## Overview

Godot provides two complementary systems for handling input: **event-driven** callbacks that respond to specific input events, and **polling** via the `Input` singleton for continuous checks. Understanding when to use each—and how input propagates through the scene tree—is essential for responsive, bug-free controls.

---

## Input Propagation Order

Input events travel through your scene in a specific order. Understanding this prevents "why isn't my input working?" bugs.

```
1. Node._input()           → First chance, catches everything
2. Control._gui_input()    → UI elements (buttons, panels, etc.)
3. Node._shortcut_input()  → Keyboard/joypad shortcuts only
4. Node._unhandled_key_input() → Unhandled keyboard events only
5. Node._unhandled_input() → Everything the GUI didn't consume
6. Physics picking         → 3D/2D collision objects
```

**Key insight:** Events propagate in **reverse tree order** (deepest children first, then parents). Any node can stop propagation by calling `get_viewport().set_input_as_handled()`.

---

## Core Input Methods

### 1. `_unhandled_input()` — Gameplay Input (Recommended)

Best for gameplay controls. Only receives events the GUI didn't consume.

```gdscript
func _unhandled_input(event: InputEvent) -> void:
    if event.is_action_pressed("jump"):
        velocity.y = -jump_force
        get_viewport().set_input_as_handled()
```

**When to use:**

- Player movement, shooting, abilities
- Camera controls
- Any gameplay input that should be blocked by UI

---

### 2. `_input()` — Global Input

Receives ALL input events before the GUI. Use sparingly.

```gdscript
func _input(event: InputEvent) -> void:
    if event.is_action_pressed("pause"):
        # Pause menu should work even when GUI has focus
        toggle_pause()
        get_viewport().set_input_as_handled()
```

**When to use:**

- System-level commands (pause, screenshot)
- Input that must work regardless of UI state
- Debug controls

---

### 3. `_gui_input()` — Control Nodes Only

Only available on Control-derived nodes. Receives mouse events within the control's rect.

```gdscript
extends Button

func _gui_input(event: InputEvent) -> void:
    if event is InputEventMouseButton and event.pressed:
        if event.button_index == MOUSE_BUTTON_RIGHT:
            show_context_menu()
            accept_event()  # Stops propagation for Controls
```

**Gotcha:** Keyboard events only reach `_gui_input()` if the Control has focus. Use `grab_focus()` or set `focus_mode`.

---

### 4. `_shortcut_input()` — Shortcuts Only

Receives only `InputEventKey`, `InputEventShortcut`, and `InputEventJoypadButton`.

```gdscript
func _shortcut_input(event: InputEvent) -> void:
    if event.is_action_pressed("quick_save"):
        save_game()
        get_viewport().set_input_as_handled()
```

---

### 5. `_unhandled_key_input()` — Keyboard Only

Like `_unhandled_input()` but filtered to keyboard events only.

```gdscript
func _unhandled_key_input(event: InputEvent) -> void:
    if event is InputEventKey and event.pressed:
        match event.keycode:
            KEY_F1:
                toggle_help()
            KEY_F11:
                toggle_fullscreen()
```

---

## Input Singleton (Polling)

Poll input state directly via the `Input` singleton. Best for continuous input like movement.

### Action State Methods

|Method|Returns `true` when...|
|---|---|
|`is_action_pressed("action")`|Action is currently held|
|`is_action_just_pressed("action")`|Action was pressed THIS frame|
|`is_action_just_released("action")`|Action was released THIS frame|

```gdscript
func _physics_process(delta: float) -> void:
    if Input.is_action_pressed("sprint"):
        speed = run_speed
    
    if Input.is_action_just_pressed("interact"):
        interact_with_nearest()
```

**Gotcha:** `is_action_just_pressed()` only works in `_process()` or `_physics_process()`. In `_input()`, use `event.is_action_pressed()` instead—the event IS the "just pressed" moment.

---

### Movement Input Methods

#### `Input.get_vector()` — 2D Movement (Circular Deadzone)

```gdscript
func _physics_process(delta: float) -> void:
    var direction := Input.get_vector("move_left", "move_right", "move_up", "move_down")
    velocity = direction * speed
```

**Behavior:**

- Returns `Vector2` with length clamped to 1.0
- Applies **circular deadzone** (good for analog sticks)
- Deadzone averaged from all four actions (or override with 5th parameter)

**Gotcha:** Circular deadzone means you can't get pure horizontal/vertical input near the deadzone threshold. For platformers needing precise cardinal directions, use `get_axis()` instead.

---

#### `Input.get_axis()` — Single Axis (Per-Axis Deadzone)

```gdscript
func _physics_process(delta: float) -> void:
    var horizontal := Input.get_axis("move_left", "move_right")
    var vertical := Input.get_axis("move_up", "move_down")
    velocity = Vector2(horizontal, vertical).limit_length() * speed
```

**When to use over `get_vector()`:**

- Platformers requiring precise left/right
- When you need per-axis deadzone behavior
- Mixing keyboard and gamepad on same actions

---

#### `Input.get_action_strength()` — Analog Value

```gdscript
var trigger_pressure := Input.get_action_strength("accelerate")
car_speed = max_speed * trigger_pressure
```

**Returns:** `0.0` to `1.0` based on how far past the deadzone the input is.

- Digital inputs (keyboard) return `0.0` or `1.0`
- Analog inputs (triggers, sticks) return intermediate values

---

#### `Input.get_action_raw_strength()` — Ignores Deadzone

Same as above but ignores the action's configured deadzone.

---

## InputEvent Types

### Type Checking Pattern

```gdscript
func _unhandled_input(event: InputEvent) -> void:
    if event is InputEventKey:
        handle_key(event)
    elif event is InputEventMouseButton:
        handle_mouse_button(event)
    elif event is InputEventMouseMotion:
        handle_mouse_motion(event)
    elif event is InputEventJoypadButton:
        handle_joypad_button(event)
    elif event is InputEventJoypadMotion:
        handle_joypad_axis(event)
```

### Common InputEvent Types

|Type|Use Case|Key Properties|
|---|---|---|
|`InputEventKey`|Keyboard|`keycode`, `physical_keycode`, `unicode`, `pressed`, `echo`|
|`InputEventMouseButton`|Mouse clicks/wheel|`button_index`, `pressed`, `double_click`, `position`|
|`InputEventMouseMotion`|Mouse movement|`relative`, `velocity`, `position`|
|`InputEventJoypadButton`|Gamepad buttons|`button_index`, `pressed`, `pressure`|
|`InputEventJoypadMotion`|Gamepad sticks/triggers|`axis`, `axis_value`|
|`InputEventScreenTouch`|Touch start/end|`index`, `position`, `pressed`|
|`InputEventScreenDrag`|Touch movement|`index`, `relative`, `velocity`, `position`|
|`InputEventAction`|Synthetic actions|`action`, `pressed`, `strength`|

---

## InputEvent Properties

### Common to All Events

```gdscript
event.device       # int: Device ID (0 for keyboard/mouse, gamepad index otherwise)
event.is_action("name")           # Check if matches action
event.is_action_pressed("name")   # Action pressed (not released)
event.is_action_released("name")  # Action released
event.is_pressed()                # Generic pressed state
event.is_released()               # Generic released state (Godot 4.3+)
event.is_echo()                   # Key repeat event
```

### InputEventKey Specifics

```gdscript
event.keycode           # Key enum (KEY_A, KEY_SPACE, etc.) - affected by layout
event.physical_keycode  # Physical key position - layout independent (WASD friendly)
event.key_label         # What the key produces with modifiers
event.unicode           # Unicode character (for text input)
event.pressed           # true if pressed, false if released
event.echo              # true if this is a key-repeat event

# Modifier checks
event.shift_pressed
event.ctrl_pressed
event.alt_pressed
event.meta_pressed      # Cmd on Mac, Win key on Windows
```

**Physical vs Logical Keycodes:**

```gdscript
# Use physical_keycode for movement (WASD position stays same on AZERTY)
if event.physical_keycode == KEY_W:
    move_forward()

# Use keycode for shortcuts (Ctrl+S is always "S" key logically)
if event.keycode == KEY_S and event.ctrl_pressed:
    save()
```

---

### InputEventMouseButton Specifics

```gdscript
event.button_index   # MOUSE_BUTTON_LEFT, _RIGHT, _MIDDLE, _WHEEL_UP, _WHEEL_DOWN, etc.
event.pressed        # true on click, false on release
event.double_click   # true if double-click
event.position       # Vector2 position in viewport
event.global_position # Vector2 position in window
event.factor         # Scroll amount (for wheel events)
```

**Gotcha:** Mouse wheel events only have `pressed = true`. There's no "release" for scroll.

```gdscript
func _unhandled_input(event: InputEvent) -> void:
    if event is InputEventMouseButton:
        match event.button_index:
            MOUSE_BUTTON_LEFT:
                if event.pressed:
                    start_shooting()
                else:
                    stop_shooting()
            MOUSE_BUTTON_WHEEL_UP:
                zoom_in()  # No need to check pressed
            MOUSE_BUTTON_WHEEL_DOWN:
                zoom_out()
```

---

### InputEventMouseMotion Specifics

```gdscript
event.relative   # Vector2: Movement since last event (use this for camera control)
event.velocity   # Vector2: Speed of mouse movement
event.position   # Vector2: Current position in viewport
event.pressure   # float: Pen pressure (graphics tablets)
event.tilt       # Vector2: Pen tilt
```

**FPS Camera Pattern:**

```gdscript
var mouse_sensitivity := 0.002

func _ready() -> void:
    Input.mouse_mode = Input.MOUSE_MODE_CAPTURED

func _unhandled_input(event: InputEvent) -> void:
    if event is InputEventMouseMotion:
        rotate_y(-event.relative.x * mouse_sensitivity)
        $Camera3D.rotate_x(-event.relative.y * mouse_sensitivity)
        $Camera3D.rotation.x = clamp($Camera3D.rotation.x, -PI/2, PI/2)
```

**Gotcha:** When `MOUSE_MODE_CAPTURED`, `event.position` is always screen center. Use `event.relative` for movement.

---

## Mouse Modes

```gdscript
Input.mouse_mode = Input.MOUSE_MODE_VISIBLE   # Normal
Input.mouse_mode = Input.MOUSE_MODE_HIDDEN    # Hidden but can leave window
Input.mouse_mode = Input.MOUSE_MODE_CAPTURED  # Hidden + locked to window (FPS games)
Input.mouse_mode = Input.MOUSE_MODE_CONFINED  # Visible but can't leave window
Input.mouse_mode = Input.MOUSE_MODE_CONFINED_HIDDEN  # Hidden + confined
```

**Common Pattern — Toggle Capture:**

```gdscript
func _unhandled_input(event: InputEvent) -> void:
    if event.is_action_pressed("ui_cancel"):
        if Input.mouse_mode == Input.MOUSE_MODE_CAPTURED:
            Input.mouse_mode = Input.MOUSE_MODE_VISIBLE
        else:
            Input.mouse_mode = Input.MOUSE_MODE_CAPTURED
```

---

## Input Actions (InputMap)

### Define in Project Settings

**Project → Project Settings → Input Map**

Or programmatically:

```gdscript
func _ready() -> void:
    # Add new action
    InputMap.add_action("shoot")
    
    # Create and configure event
    var event := InputEventMouseButton.new()
    event.button_index = MOUSE_BUTTON_LEFT
    
    # Bind event to action
    InputMap.action_add_event("shoot", event)
    
    # Set deadzone (for analog actions)
    InputMap.action_set_deadzone("move_left", 0.2)
```

### Remove Actions

```gdscript
InputMap.action_erase_events("shoot")  # Remove all bindings
InputMap.erase_action("shoot")         # Remove action entirely
```

---

## Control Node Input Filtering

Control nodes can intercept mouse input before it reaches game objects.

### mouse_filter Property

```gdscript
# Receives mouse events AND blocks them from going further
control.mouse_filter = Control.MOUSE_FILTER_STOP

# Receives mouse events but lets them pass through
control.mouse_filter = Control.MOUSE_FILTER_PASS

# Ignores mouse events entirely (click-through)
control.mouse_filter = Control.MOUSE_FILTER_IGNORE
```

**Common Issue:** Can't click game objects behind UI?

```gdscript
# On your HUD/overlay containers:
$HealthBar.mouse_filter = Control.MOUSE_FILTER_IGNORE
$Minimap.mouse_filter = Control.MOUSE_FILTER_IGNORE
```

---

## Stopping Input Propagation

|Context|Method|
|---|---|
|Any `_input` callback|`get_viewport().set_input_as_handled()`|
|Control `_gui_input`|`accept_event()`|
|Check if handled|`get_viewport().is_input_handled()`|

```gdscript
func _unhandled_input(event: InputEvent) -> void:
    if event.is_action_pressed("interact"):
        if try_interact():  # Returns true if interaction happened
            get_viewport().set_input_as_handled()
```

---

## Enabling/Disabling Input Processing

```gdscript
# Per-callback control
set_process_input(false)                 # Disable _input()
set_process_unhandled_input(false)       # Disable _unhandled_input()
set_process_unhandled_key_input(false)   # Disable _unhandled_key_input()
set_process_shortcut_input(false)        # Disable _shortcut_input()

# Check status
is_processing_input()
is_processing_unhandled_input()
```

---

## Common Patterns

### Action-Based Input (Recommended)

```gdscript
# In _physics_process for continuous checks
func _physics_process(delta: float) -> void:
    var input_dir := Input.get_vector("left", "right", "up", "down")
    velocity = input_dir * speed
    
    if Input.is_action_pressed("sprint"):
        velocity *= 1.5
    
    move_and_slide()

# In _unhandled_input for one-shot actions
func _unhandled_input(event: InputEvent) -> void:
    if event.is_action_pressed("jump") and is_on_floor():
        velocity.y = jump_force
```

---

### Input Buffering (Coyote Time / Jump Buffer)

```gdscript
var jump_buffer_time := 0.1
var coyote_time := 0.1
var _jump_buffer_timer := 0.0
var _coyote_timer := 0.0
var _was_on_floor := false

func _physics_process(delta: float) -> void:
    # Coyote time
    if is_on_floor():
        _coyote_timer = coyote_time
    else:
        _coyote_timer -= delta
    
    # Jump buffer
    if Input.is_action_just_pressed("jump"):
        _jump_buffer_timer = jump_buffer_time
    else:
        _jump_buffer_timer -= delta
    
    # Execute jump if either timer valid
    if _jump_buffer_timer > 0 and _coyote_timer > 0:
        velocity.y = jump_force
        _jump_buffer_timer = 0
        _coyote_timer = 0
    
    move_and_slide()
```

---

### Rebindable Controls

```gdscript
func rebind_action(action_name: String, new_event: InputEvent) -> void:
    InputMap.action_erase_events(action_name)
    InputMap.action_add_event(action_name, new_event)

func get_action_key_name(action_name: String) -> String:
    var events := InputMap.action_get_events(action_name)
    for event in events:
        if event is InputEventKey:
            return event.as_text()
    return "Unbound"

# Wait for player input to rebind
func _unhandled_input(event: InputEvent) -> void:
    if waiting_for_rebind:
        if event is InputEventKey or event is InputEventMouseButton:
            rebind_action(action_to_rebind, event)
            waiting_for_rebind = false
            get_viewport().set_input_as_handled()
```

---

### Synthetic Input Events

```gdscript
# Simulate a key press
func simulate_jump() -> void:
    var event := InputEventAction.new()
    event.action = "jump"
    event.pressed = true
    Input.parse_input_event(event)

# Alternative: Use action methods directly
Input.action_press("jump")
# ... later ...
Input.action_release("jump")
```

**Note:** `Input.action_press()` won't trigger `_input()` callbacks—it only affects `Input.is_action_pressed()` polling. Use `parse_input_event()` for full simulation.

---

## Quick Reference Table

| Goal                         | Method                                                          |
| ---------------------------- | --------------------------------------------------------------- |
| Gameplay input (respects UI) | `_unhandled_input(event)`                                       |
| System commands (ignores UI) | `_input(event)`                                                 |
| UI-specific input            | `_gui_input(event)`                                             |
| Continuous movement          | `Input.get_vector()` / `Input.get_axis()` in `_physics_process` |
| One-shot action (pressed)    | `event.is_action_pressed("action")` in callback                 |
| One-shot action (polling)    | `Input.is_action_just_pressed("action")` in process             |
| Action held down             | `Input.is_action_pressed("action")`                             |
| Analog input strength        | `Input.get_action_strength("action")`                           |
| Stop event propagation       | `get_viewport().set_input_as_handled()`                         |
| Stop in Control nodes        | `accept_event()`                                                |
| Check event type             | `event is InputEventKey`                                        |
| Mouse camera control         | `event.relative` from `InputEventMouseMotion`                   |
| Lock mouse to window         | `Input.mouse_mode = Input.MOUSE_MODE_CAPTURED`                  |
| Make UI click-through        | `control.mouse_filter = MOUSE_FILTER_IGNORE`                    |

---

## Debugging Input

```gdscript
# Print all input events
func _input(event: InputEvent) -> void:
    print(event.as_text())

# Check what's bound to an action
func print_bindings(action: String) -> void:
    for event in InputMap.action_get_events(action):
        print(event.as_text())

# Visualize input in-game
func _process(delta: float) -> void:
    $DebugLabel.text = "Move: %s\nJump: %s" % [
        Input.get_vector("left", "right", "up", "down"),
        Input.is_action_pressed("jump")
    ]
```

---

## Project Settings

**Project → Project Settings → Input Devices**

| Setting                             | Description                                |
| ----------------------------------- | ------------------------------------------ |
| `pointing/emulate_touch_from_mouse` | Mouse clicks generate touch events         |
| `pointing/emulate_mouse_from_touch` | Touch generates mouse events (default: on) |
| `buffering/agile_event_flushing`    | Reduce input latency (may increase CPU)    |

**Input Map Settings (per-action):**

- **Deadzone:** Threshold before analog input registers (0.0-1.0)