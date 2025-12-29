# EditorPlugin System

## Why You Need It

`@tool` scripts cannot receive viewport input—the editor consumes it first. For click-to-place, custom gizmos, or viewport interaction, you need an EditorPlugin.

## Architecture Overview

|Component|EditorPlugin|@tool Node|
|---|---|---|
|Lives in|Addon system|Scene tree|
|User selects it|No|Yes|
|@export visible|No|Yes|
|Handles input|Yes|No|
|Stores settings|No (use EditorSettings)|Yes|
|Lifecycle|Plugin enable/disable|Scene open/close|

Typical pattern: Plugin handles editor interaction, node stores settings and logic.

## Folder Structure

```
res://addons/your_plugin_name/
    plugin.cfg              # Required: plugin metadata
    your_plugin.gd          # EditorPlugin script
    your_tool.gd            # Your @tool node script
    icon.svg                # Optional: icon for node
```

## plugin.cfg

Create this as a plain text file (not via Godot's script dialog):

```ini
[plugin]

name="Your Plugin Name"
description="What it does"
author="Your Name"
version="1.0"
script="your_plugin.gd"
```

## Basic EditorPlugin

```gdscript
@tool
extends EditorPlugin

func _enter_tree():
    # Plugin enabled - setup here
    print("Plugin loaded")

func _exit_tree():
    # Plugin disabled - cleanup here
    print("Plugin unloaded")
```

Note: `@tool` is still required on EditorPlugin scripts.

## Enabling the Plugin

1. Create the addon folder structure with `plugin.cfg`
2. Go to Project → Project Settings → Plugins
3. Find your plugin in the list
4. Check the "Enable" checkbox

## Selection-Based Activation

Make your plugin respond to node selection:

```gdscript
@tool
extends EditorPlugin

var active_tool: YourToolNode

# Editor asks: "Do you handle this type?"
func _handles(object) -> bool:
    return object is YourToolNode

# Editor says: "User selected this node"
func _edit(object):
    active_tool = object

# Editor says: "User deselected / selected something else"
func _make_visible(visible: bool):
    if not visible:
        active_tool = null
```

Flow:

1. User clicks YourToolNode in scene tree
2. Editor calls `_handles(node)` on all plugins
3. Your plugin returns `true`
4. Editor calls `_edit(node)` → store the reference
5. User clicks something else
6. Editor calls `_make_visible(false)` → clear the reference

## Handling Viewport Input

### 3D Viewport

```gdscript
func _forward_3d_gui_input(camera: Camera3D, event: InputEvent) -> int:
    if not active_tool:
        return AFTER_GUI_INPUT_PASS
    
    if event is InputEventMouseButton:
        if event.button_index == MOUSE_BUTTON_LEFT and event.pressed:
            # Handle click
            return AFTER_GUI_INPUT_STOP  # Consume the event
    
    return AFTER_GUI_INPUT_PASS  # Let editor handle it
```

### 2D Viewport

```gdscript
func _forward_canvas_gui_input(event: InputEvent) -> bool:
    if event is InputEventMouseButton and event.pressed:
        # Handle click
        return true  # Consume the event
    return false  # Let editor handle it
```

### Return Values

For 3D (`_forward_3d_gui_input`):

- `AFTER_GUI_INPUT_PASS` (0) - Let editor process the event
- `AFTER_GUI_INPUT_STOP` (1) - Consume the event, editor won't see it

For 2D (`_forward_canvas_gui_input`):

- `false` - Let editor process the event
- `true` - Consume the event

## Detecting Press vs Hold

```gdscript
func _forward_3d_gui_input(camera, event) -> int:
    if event is InputEventMouseButton:
        if event.button_index == MOUSE_BUTTON_LEFT:
            if event.pressed:
                # Mouse button just pressed (single trigger)
                _on_click(camera, event.position)
            else:
                # Mouse button just released
                pass
            return AFTER_GUI_INPUT_STOP
    
    return AFTER_GUI_INPUT_PASS
```

- `event.pressed == true` → button just went down
- `event.pressed == false` → button just released

`InputEventMouseMotion` fires continuously while moving—don't confuse it with button events.

## Preventing Selection Change on Click

When `marking_mode` is on, consume clicks to prevent selecting other objects:

```gdscript
func _forward_3d_gui_input(camera, event) -> int:
    if not active_tool or not active_tool.marking_mode:
        return AFTER_GUI_INPUT_PASS
    
    if event is InputEventMouseButton and event.button_index == MOUSE_BUTTON_LEFT:
        if event.pressed:
            _place_object(camera, event.position)
        return AFTER_GUI_INPUT_STOP  # Always consume while marking
    
    return AFTER_GUI_INPUT_PASS
```

## Adding Toolbar Buttons

```gdscript
@tool
extends EditorPlugin

var button: Button

func _enter_tree():
    button = Button.new()
    button.text = "Add Tool"
    button.pressed.connect(_on_button_pressed)
    add_control_to_container(CONTAINER_SPATIAL_EDITOR_MENU, button)

func _exit_tree():
    if button:
        button.queue_free()

func _on_button_pressed():
    var scene = preload("res://addons/your_plugin/your_tool.tscn")
    var instance = scene.instantiate()
    var root = get_editor_interface().get_edited_scene_root()
    root.add_child(instance)
    instance.owner = root  # Required for saving
```

### Container Locations

- `CONTAINER_SPATIAL_EDITOR_MENU` - 3D editor toolbar
- `CONTAINER_CANVAS_EDITOR_MENU` - 2D editor toolbar
- `CONTAINER_TOOLBAR` - Main toolbar
- `CONTAINER_SPATIAL_EDITOR_SIDE_LEFT` - Left side of 3D viewport
- `CONTAINER_SPATIAL_EDITOR_SIDE_RIGHT` - Right side of 3D viewport

## Adding Dock Panels

```gdscript
var dock: Control

func _enter_tree():
    dock = preload("res://addons/your_plugin/dock.tscn").instantiate()
    add_control_to_dock(DOCK_SLOT_RIGHT_UL, dock)

func _exit_tree():
    remove_control_from_docks(dock)
    dock.queue_free()
```

## Registering Custom Node Types

```gdscript
func _enter_tree():
    add_custom_type(
        "YourTool",                                      # Name in Add Node dialog
        "Node3D",                                        # Base type
        preload("res://addons/your_plugin/your_tool.gd"), # Script
        preload("res://addons/your_plugin/icon.svg")     # Icon (optional)
    )

func _exit_tree():
    remove_custom_type("YourTool")
```

Note: This only registers the script, not a scene with children. For scenes with children, use the toolbar button approach.

## Accessing Editor Interfaces

```gdscript
# Get the root of the currently edited scene
var root = get_editor_interface().get_edited_scene_root()

# Get the editor's selection
var selection = get_editor_interface().get_selection()
var selected_nodes = selection.get_selected_nodes()

# Get editor settings
var settings = get_editor_interface().get_editor_settings()
```

## Complete Example: Scatter Tool Plugin

```gdscript
# scatter_plugin.gd
@tool
extends EditorPlugin

var active_tool: ScatterTool
var add_button: Button

func _enter_tree():
    # Register custom type
    add_custom_type(
        "ScatterTool",
        "Node3D",
        preload("res://addons/scatter_tool/scatter_tool.gd"),
        null
    )
    
    # Add toolbar button
    add_button = Button.new()
    add_button.text = "Add ScatterTool"
    add_button.pressed.connect(_add_scatter_tool)
    add_control_to_container(CONTAINER_SPATIAL_EDITOR_MENU, add_button)

func _exit_tree():
    remove_custom_type("ScatterTool")
    if add_button:
        add_button.queue_free()

func _handles(object) -> bool:
    return object is ScatterTool

func _edit(object):
    active_tool = object

func _make_visible(visible: bool):
    if not visible:
        active_tool = null

func _forward_3d_gui_input(camera: Camera3D, event: InputEvent) -> int:
    if not active_tool or not active_tool.marking_mode:
        return AFTER_GUI_INPUT_PASS
    
    if event is InputEventMouseButton:
        if event.button_index == MOUSE_BUTTON_LEFT and event.pressed:
            var result = _raycast(camera, event.position)
            if result:
                active_tool.place_at(result.position, result.normal)
            return AFTER_GUI_INPUT_STOP
    
    return AFTER_GUI_INPUT_PASS

func _raycast(camera: Camera3D, screen_pos: Vector2) -> Dictionary:
    var from = camera.project_ray_origin(screen_pos)
    var dir = camera.project_ray_normal(screen_pos)
    var to = from + dir * 1000.0
    
    var space = camera.get_world_3d().direct_space_state
    var query = PhysicsRayQueryParameters3D.create(from, to)
    return space.intersect_ray(query)

func _add_scatter_tool():
    var tool = ScatterTool.new()
    var root = get_editor_interface().get_edited_scene_root()
    root.add_child(tool)
    tool.owner = root
```

```gdscript
# scatter_tool.gd
@tool
class_name ScatterTool
extends Node3D

@export var marking_mode: bool = false
@export var mesh_to_scatter: PackedScene

func _enter_tree():
    if Engine.is_editor_hint():
        _ensure_children.call_deferred()

func _ensure_children():
    if not has_node("Generated"):
        var node = Node3D.new()
        node.name = "Generated"
        add_child(node)
        node.owner = get_tree().edited_scene_root

func place_at(position: Vector3, normal: Vector3):
    if not mesh_to_scatter:
        push_warning("No mesh assigned to scatter")
        return
    
    var instance = mesh_to_scatter.instantiate()
    $Generated.add_child(instance)
    instance.owner = get_tree().edited_scene_root
    instance.global_position = position
    
    # Align to normal (optional)
    if normal != Vector3.UP:
        instance.look_at(position + normal, Vector3.UP)
        instance.rotate_object_local(Vector3.RIGHT, PI / 2)
```

---

# EditorPlugin Essentials

```gdscript
_enter_tree()                      # Plugin enabled
_exit_tree()                       # Plugin disabled
_handles(object) -> bool           # "Do I handle this type?"
_edit(object)                      # "Here's the selected object"
_make_visible(visible)             # "Selection changed"
_forward_3d_gui_input(cam, event)  # Viewport input
AFTER_GUI_INPUT_PASS               # Let editor handle event
AFTER_GUI_INPUT_STOP               # Consume event
```

