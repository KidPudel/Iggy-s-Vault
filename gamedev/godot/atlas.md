# Godot AtlasTexture Reference

## What is AtlasTexture?

AtlasTexture is a texture that references a rectangular region from a larger texture (texture atlas/spritesheet). Instead of having 100 separate image files, you pack them into one atlas and use AtlasTexture to slice out individual sprites.

## Why Use Texture Atlases?

✅ **Performance:** Fewer texture swaps = faster rendering  
✅ **Memory:** Better GPU memory utilization  
✅ **Organization:** One file instead of hundreds  
✅ **Load times:** Faster to load one large image than many small ones

## Basic Setup

### In Editor

1. Import your spritesheet
2. Create new AtlasTexture resource
3. Set `atlas` property to your spritesheet
4. Set `region` to define the slice (x, y, width, height)

### In Code

```gdscript
var atlas = AtlasTexture.new()
atlas.atlas = preload("res://spritesheet.png")
atlas.region = Rect2(0, 0, 32, 32)  # x, y, width, height

$Sprite2D.texture = atlas
```

## Key Properties

```gdscript
# The source texture (your spritesheet)
atlas.atlas = texture

# The rectangular region to extract
atlas.region = Rect2(x, y, width, height)

# Margin around the region (rarely used)
atlas.margin = Rect2(left, top, right, bottom)

# Whether to apply texture filtering
atlas.filter_clip = false
```

## Common Patterns

### 1. Slicing a Grid-Based Spritesheet

```gdscript
# Spritesheet with 16x16 tiles in a 10x10 grid
const TILE_SIZE = 16

func get_tile_texture(tile_x: int, tile_y: int) -> AtlasTexture:
    var atlas = AtlasTexture.new()
    atlas.atlas = preload("res://tileset.png")
    atlas.region = Rect2(
        tile_x * TILE_SIZE,
        tile_y * TILE_SIZE,
        TILE_SIZE,
        TILE_SIZE
    )
    return atlas

# Usage
$Sprite2D.texture = get_tile_texture(3, 5)  # Tile at column 3, row 5
```

### 2. Animation Frames from Spritesheet

```gdscript
# Character spritesheet: 4 frames of 32x32 animation in a row
var frames: Array[AtlasTexture] = []

func load_animation_frames():
    var spritesheet = preload("res://character.png")
    
    for i in range(4):
        var atlas = AtlasTexture.new()
        atlas.atlas = spritesheet
        atlas.region = Rect2(i * 32, 0, 32, 32)
        frames.append(atlas)
    
    # Use with AnimatedSprite2D
    var sprite_frames = SpriteFrames.new()
    sprite_frames.add_animation("run")
    for frame in frames:
        sprite_frames.add_frame("run", frame)
    
    $AnimatedSprite2D.sprite_frames = sprite_frames
```

### 3. Dynamic Item Icons

```gdscript
# Icon atlas: 64x64 icons in 8x8 grid
const ICON_SIZE = 64
var icon_atlas = preload("res://ui/icons_atlas.png")

func get_item_icon(item_id: int) -> AtlasTexture:
    var atlas = AtlasTexture.new()
    atlas.atlas = icon_atlas
    
    var icons_per_row = 8
    var row = item_id / icons_per_row
    var col = item_id % icons_per_row
    
    atlas.region = Rect2(
        col * ICON_SIZE,
        row * ICON_SIZE,
        ICON_SIZE,
        ICON_SIZE
    )
    return atlas

# Usage
$ItemSlot.texture = get_item_icon(15)  # Get icon #15
```

### 4. UI Nine-Patch Alternative

```gdscript
# Sometimes simpler than NinePatchRect for basic borders
func create_border_corner() -> AtlasTexture:
    var atlas = AtlasTexture.new()
    atlas.atlas = preload("res://ui/borders.png")
    atlas.region = Rect2(0, 0, 8, 8)  # Top-left corner
    return atlas
```

## Working with TextureRect

```gdscript
var texture_rect = TextureRect.new()
texture_rect.texture = get_tile_texture(2, 3)
texture_rect.expand_mode = TextureRect.EXPAND_FIT_WIDTH
texture_rect.stretch_mode = TextureRect.STRETCH_KEEP_ASPECT
```

## Performance Tips

### ✅ Do This

```gdscript
# Cache atlas textures, don't recreate every frame
var cached_textures: Dictionary = {}

func get_cached_tile(x: int, y: int) -> AtlasTexture:
    var key = Vector2i(x, y)
    if not cached_textures.has(key):
        cached_textures[key] = create_tile_texture(x, y)
    return cached_textures[key]
```

### ❌ Don't Do This

```gdscript
# Creating new AtlasTexture every frame = bad
func _process(delta):
    var atlas = AtlasTexture.new()  # Memory leak!
    atlas.atlas = spritesheet
    atlas.region = Rect2(...)
    $Sprite2D.texture = atlas
```

## AtlasTexture vs. Sprite Frames

**Use AtlasTexture when:**

- Manually slicing static sprites
- Building UI from atlas
- Custom animation systems
- Procedural texture selection

**Use SpriteFrames when:**

- AnimatedSprite2D animations
- Editor wants to preview animations
- Non-programmers need to configure animations

## Common Gotchas

### Region Outside Texture Bounds

```gdscript
# This will show nothing or errors
atlas.region = Rect2(1000, 1000, 32, 32)  # Outside texture!

# Always validate:
var texture_size = atlas.atlas.get_size()
if region.position.x + region.size.x <= texture_size.x:
    atlas.region = region
```

### Pixel Bleeding

If you see thin lines from adjacent sprites:

```gdscript
# Add 1px padding between sprites in your atlas
# Or use margin property:
atlas.margin = Rect2(1, 1, 1, 1)

# Or ensure filtering is off:
atlas.atlas.set_flags(Texture2D.FLAG_FILTER)  # Godot 3.x
# In Godot 4.x, set in import settings
```

### Memory Management

```gdscript
# AtlasTexture doesn't duplicate the source texture
# All atlas instances share the same base texture (good!)
var atlas1 = create_atlas()
var atlas2 = create_atlas()
# atlas1.atlas and atlas2.atlas point to same texture in memory
```

## Debug Visualization

```gdscript
func debug_atlas_region(atlas: AtlasTexture):
    print("Atlas: %s" % atlas.atlas.resource_path)
    print("Region: %s" % atlas.region)
    print("Size: %dx%d" % [atlas.region.size.x, atlas.region.size.y])
```

## Quick Example: Inventory Grid

```gdscript
extends GridContainer

const ICON_SIZE = 64
var item_icons = preload("res://items_atlas.png")

func add_item_slot(item_id: int):
    var texture_rect = TextureRect.new()
    texture_rect.custom_minimum_size = Vector2(ICON_SIZE, ICON_SIZE)
    
    var atlas = AtlasTexture.new()
    atlas.atlas = item_icons
    atlas.region = Rect2(
        (item_id % 8) * ICON_SIZE,
        (item_id / 8) * ICON_SIZE,
        ICON_SIZE,
        ICON_SIZE
    )
    
    texture_rect.texture = atlas
    add_child(texture_rect)
```

## Key Takeaway

AtlasTexture is essentially a "window" into a larger texture. It's a lightweight reference, not a copy, making it perfect for working with spritesheets and texture atlases efficiently.