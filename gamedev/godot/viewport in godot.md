# Godot Viewport Reference

In Godot [[viewport]] is both:

- Rendering target - surface where scenes are drawn
- A node (Viewport) - container that renders its own scene tree, isolated from main scene tree

## Getting Viewports

`some_ref_to_node.get_viewport()` - returns the closest `Viewport` ancestor `get_tree().root` - returns the root viewport (main window)

Each viewport is its own rendering target. You can render a whole scene tree into SubViewport, assign it to a texture outside of it, and get a flattened texture.

## Viewport Types

**Root Viewport** - The main game window, automatically created **SubViewport** - Child viewport node, renders independently

---

# SubViewport Deep Dive

## What It Is

SubViewport is a node that creates an isolated rendering context. It has its own scene tree and renders completely independently from the parent viewport. Think of it as a "render to texture" node.

## Key Difference from Root Viewport

- Root viewport = your game window, created automatically
- SubViewport = manually created, renders to texture, not directly visible
- SubViewport **requires explicit size** - doesn't inherit from parent
- SubViewport has **no automatic input handling** unless using SubViewportContainer

## Common Properties

```gdscript
viewport.size                    # Vector2i - viewport dimensions
viewport.get_visible_rect()      # Rect2 - visible area
viewport.get_texture()           # ViewportTexture - rendered output
```

## SubViewport Configuration

**Render Target Update Mode:**

- `UPDATE_ALWAYS` - renders every frame (use for real-time)
- `UPDATE_ONCE` - renders single frame then stops
- `UPDATE_WHEN_VISIBLE` - only when visible in tree
- `UPDATE_WHEN_PARENT_VISIBLE` - only when parent is visible
- `UPDATE_DISABLED` - never updates

**Critical Properties:**

```gdscript
size: Vector2i                    # MUST set manually, no default inheritance
transparent_bg: bool              # Clear to transparent vs clear_color
render_target_clear_mode: int     # How to clear between frames
render_target_update_mode: int    # When to render (see above)
handle_input_locally: bool        # Process input events in SubViewport
audio_listener_enable_2d: bool    # Make this the 2D audio listener
audio_listener_enable_3d: bool    # Make this the 3D audio listener
own_world_3d: bool                # Use separate World3D (physics isolation)
```

**Common Gotchas:**

- Default size is (512, 512) - almost never what you want
- No automatic resize when window changes
- `transparent_bg = true` can cause performance hit
- Input doesn't work unless using SubViewportContainer or manual forwarding

## SubViewportContainer

**Purpose:** Optional parent node that provides automatic features for SubViewport

**What it does:**

- Automatically forwards mouse/touch input to SubViewport
- Stretches SubViewport to fill container size (if stretch enabled)
- Handles input scaling based on SubViewport vs container size

**When to use:**

- You need mouse interaction with SubViewport content
- You want automatic size management
- Building UI with interactive viewport content (minimap with clicks, etc)

**When NOT to use:**

- Post-processing pipelines (use TextureRect instead)
- Just displaying viewport output without interaction
- You're manually managing size and input

**Structure with container:**

```
SubViewportContainer
└─ SubViewport
   └─ Your scene content
```

**Structure without container (post-processing):**

```
Root
├─ SubViewport (your 3D scene)
└─ TextureRect (displays get_texture(), applies shader)
```

Container is **optional** - don't use it unless you need its features.

### Post-Processing Pipeline

```
SubViewport (3D scene)
  ├─ Camera3D
  └─ Your 3D nodes
  
TextureRect (outside viewport)
  └─ ShaderMaterial (post-process effect)
```

```gdscript
# Setup
$TextureRect.texture = $SubViewport.get_texture()
$SubViewport.size = get_viewport().get_visible_rect().size
```

### Minimap/Picture-in-Picture

Render same scene from different camera angle in small SubViewport

### UI Rendering

Render 3D objects in UI (character preview, item icons)

### Render-to-Texture

Generate textures at runtime (portals, mirrors, security cameras)

## Size Management

SubViewport size **never** inherits automatically - you must set it:

```gdscript
func _ready():
    _resize_viewport()
    get_tree().root.size_changed.connect(_resize_viewport)

func _resize_viewport():
    $SubViewport.size = get_viewport().get_visible_rect().size
    # Or use custom size for performance:
    # $SubViewport.size = Vector2i(1280, 720)
```

**Performance tip:** Use lower resolution than window for expensive effects:

```gdscript
# Half resolution for blur/glow effects
$SubViewport.size = get_viewport().get_visible_rect().size / 2
```

## Input Handling in SubViewport

Input doesn't automatically work in SubViewport. Three options:

**Option 1: Use SubViewportContainer** (easiest)

```
SubViewportContainer
└─ SubViewport
```

Automatically forwards input events.

**Option 2: Manual forwarding**

```gdscript
func _input(event):
    if event is InputEventMouse:
        # Transform mouse position to SubViewport space
        var local_event = event.xformed_by(transform)
        $SubViewport.push_input(local_event)
```

**Option 3: `handle_input_locally = false`** SubViewport receives input from parent viewport's camera (only works for 3D).

For most cases: if you need input, use SubViewportContainer. If you don't need input (post-processing), skip it.

- Each SubViewport is a full render pass - expensive
- Use lower resolution SubViewports for effects that don't need full detail
- Disable when not needed via `render_target_update_mode`
- Multiple SubViewports multiply rendering cost

## Common SubViewport Gotchas

- **Size defaults to (512, 512)** - always set it explicitly, it doesn't inherit
- **`SCREEN_TEXTURE` in shaders** references what's _behind_ the node in scene tree, NOT the TextureRect's texture
- **Use `TEXTURE` built-in** in canvas_item shaders to sample the TextureRect's assigned texture
- **No size_changed signal** - listen to root viewport's signal instead
- **ViewportTexture updates automatically** - get_texture() gives you live-updating texture
- **Input doesn't work by default** - use SubViewportContainer or forward manually
- **Each SubViewport = full render pass** - don't create dozens of them
- **transparent_bg has cost** - only enable if you actually need transparency
- **World3D separation** - if `own_world_3d = true`, physics and environment are isolated