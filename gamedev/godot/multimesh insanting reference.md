# MultiMesh Instancing in Godot

#godot #optimization #3d #instancing #tool

## The Problem

Rendering many identical objects (trees, grass, rocks) individually is expensive:

```
1,000 trees = 1,000 draw calls = GPU bottleneck
```

Each draw call has CPU overhead. The GPU spends more time waiting than rendering.

---

## The Solution: MultiMesh

MultiMesh batches identical meshes into **one draw call**:

```
1,000 trees = 1 draw call = fast
```

The GPU renders all instances in a single batch using the same mesh and material.

---

## Core Concepts

### What MultiMesh Handles

| ✅ Handles                      | ❌ Does NOT Handle  |
| ------------------------------ | ------------------ |
| Mesh rendering                 | Collisions         |
| Per-instance transforms        | Scripts            |
| Per-instance color/custom data | Signals            |
| Material                       | Child nodes        |
| Shadows (as single batch)      | Individual culling |

### Node Structure

```
MultiMeshInstance3D          ← The node you add to scene
└── MultiMesh (resource)     ← Contains instance data
    └── Mesh (resource)      ← The actual mesh to repeat
```

---

## Basic Implementation

### Minimal Example

```gdscript
extends Node3D

func _ready():
    var mm = MultiMesh.new()
    mm.mesh = preload("res://tree.tres")
    mm.transform_format = MultiMesh.TRANSFORM_3D
    mm.instance_count = 100
    
    for i in 100:
        var xform = Transform3D()
        xform.origin = Vector3(randf() * 50, 0, randf() * 50)
        mm.set_instance_transform(i, xform)
    
    var mmi = MultiMeshInstance3D.new()
    mmi.multimesh = mm
    add_child(mmi)
```

### With Rotation and Scale

```gdscript
for i in instance_count:
    var xform = Transform3D()
    
    # Random rotation around Y
    xform = xform.rotated(Vector3.UP, randf() * TAU)
    
    # Random scale
    var scale = randf_range(0.8, 1.2)
    xform = xform.scaled(Vector3.ONE * scale)
    
    # Position
    xform.origin = positions[i]
    
    mm.set_instance_transform(i, xform)
```

---

## The Culling Problem

### How Frustum Culling Works

```
        Camera View
         ┌─────┐
        ╱       ╲
       ╱    ○    ╲      ← Rendered (in view)
      ╱           ╲
     ╱             ╲
    ╱   ○       ○   ╲   ← Rendered (in view)
   ╱                 ╲
  ────────────────────
         ○   ○          ← NOT rendered (outside view)
```

Objects outside the camera's view frustum are skipped.

### The MultiMesh Catch

MultiMesh is treated as **one object** for culling:

```
┌─────────────────────────────────────────────┐
│                 ONE MultiMesh               │
│   ○   ○   ○   ○   ○   ○   ○   ○   ○   ○    │
│   ○   ○   ○   ○   ○   ○   ○   ○   ○   ○    │
│   ○   ○   ○   ○   ○   ○   ○   ○   ○   ○    │
└─────────────────────────────────────────────┘
         ▲
    Camera sees ONE tree?
    ALL trees get rendered.
```

> [!warning] Large Area = Bad Performance A single MultiMesh spanning your entire world will render ALL instances even if only one is visible.

---

## The Solution: Chunking

Split the world into grid cells, each with its own MultiMesh:

```
┌──────────┬──────────┬──────────┐
│ Chunk    │ Chunk    │ Chunk    │
│ (0,0)    │ (1,0)    │ (2,0)    │
│  ○ ○ ○   │  ○ ○ ○   │  ○ ○ ○   │
├──────────┼──────────┼──────────┤
│ Chunk    │ Chunk    │ Chunk    │
│ (0,1)    │ (1,1)    │ (2,1)    │
│  ○ ○ ○   │  ○ ○ ○   │  ○ ○ ○   │
└──────────┴──────────┴──────────┘

Camera sees Chunk (1,1)?
Only that chunk renders.
Other chunks culled entirely.
```

### Chunk Size Guidelines

|Size|Result|
|---|---|
|Too small (5m)|Too many draw calls, defeats batching|
|Too large (500m)|Back to the culling problem|
|Sweet spot (32-64m)|Good balance for most games|

Rule of thumb: chunk size ≈ typical view distance or slightly smaller.

### Chunking Code

```gdscript
func sort_into_chunks(positions: Array[Vector3], chunk_size: float) -> Dictionary:
    var chunks: Dictionary = {}  # Vector2i -> Array[Vector3]
    
    for pos in positions:
        var coord = Vector2i(
            floori(pos.x / chunk_size),
            floori(pos.z / chunk_size)
        )
        if coord not in chunks:
            chunks[coord] = []
        chunks[coord].append(pos)
    
    return chunks

func create_chunk_multimesh(positions: Array, mesh: Mesh, chunk_name: String) -> MultiMeshInstance3D:
    var mm = MultiMesh.new()
    mm.mesh = mesh
    mm.transform_format = MultiMesh.TRANSFORM_3D
    mm.instance_count = positions.size()
    
    for i in positions.size():
        var xform = Transform3D()
        xform.origin = positions[i]
        mm.set_instance_transform(i, xform)
    
    var mmi = MultiMeshInstance3D.new()
    mmi.multimesh = mm
    mmi.name = chunk_name
    return mmi
```

---

## Handling Collisions

MultiMesh doesn't do collision. You need a separate system.

### Option 1: Static Collisions (Simple)

Create collision bodies alongside the MultiMesh:

```gdscript
func create_collisions(positions: Array[Vector3], radius: float, height: float) -> Node3D:
    var container = Node3D.new()
    container.name = "Collisions"
    
    # Shared shape resource (memory efficient)
    var shape = CylinderShape3D.new()
    shape.radius = radius
    shape.height = height
    
    for i in positions.size():
        var body = StaticBody3D.new()
        var col = CollisionShape3D.new()
        col.shape = shape  # Reuse same shape
        body.add_child(col)
        container.add_child(body)
        body.position = positions[i] + Vector3(0, height / 2.0, 0)
    
    return container
```

**Pros:** Simple, always works **Cons:** Memory cost scales with instance count

### Option 2: Proximity Pooling (Scalable)

Only spawn collisions near the player:

```gdscript
extends Node3D

var all_positions: Array[Vector3] = []
var active_collisions: Dictionary = {}  # position_hash -> StaticBody3D
var collision_pool: Array[StaticBody3D] = []

@export var collision_radius: float = 30.0
@export var pool_size: int = 50

func _ready():
    _create_pool()

func _create_pool():
    var shape = CylinderShape3D.new()
    shape.radius = 0.3
    shape.height = 2.0
    
    for i in pool_size:
        var body = StaticBody3D.new()
        var col = CollisionShape3D.new()
        col.shape = shape
        body.add_child(col)
        body.position = Vector3(0, -1000, 0)  # Hide initially
        add_child(body)
        collision_pool.append(body)

func _physics_process(_delta):
    var player_pos = _get_player_position()
    _update_collisions(player_pos)

func _update_collisions(player_pos: Vector3):
    var needed_positions: Array[Vector3] = []
    
    # Find positions needing collision
    for pos in all_positions:
        if pos.distance_to(player_pos) < collision_radius:
            needed_positions.append(pos)
    
    # Return unneeded collisions to pool
    for pos_hash in active_collisions.keys():
        var pos = _unhash(pos_hash)
        if pos not in needed_positions:
            var body = active_collisions[pos_hash]
            body.position = Vector3(0, -1000, 0)
            collision_pool.append(body)
            active_collisions.erase(pos_hash)
    
    # Activate collisions for nearby positions
    for pos in needed_positions:
        var pos_hash = _hash(pos)
        if pos_hash not in active_collisions and collision_pool.size() > 0:
            var body = collision_pool.pop_back()
            body.position = pos + Vector3(0, 1, 0)
            active_collisions[pos_hash] = body

func _hash(pos: Vector3) -> int:
    return hash(pos)

func _unhash(h: int) -> Vector3:
    # Store original positions if needed
    return Vector3.ZERO
```

**Pros:** Scales to millions of instances **Cons:** More complex, slight overhead

### Option 3: Baked Collision Mesh

For fixed layouts, merge all collision shapes in Blender:

1. Create cylinders at each trunk position in Blender
2. Join into single mesh (`Ctrl+J`)
3. Export as collision-only GLB
4. Import to Godot as `ConcavePolygonShape3D`

**Pros:** Single collision body, very efficient **Cons:** Can't modify at runtime

### Which to Use?

|Instance Count|Approach|
|---|---|
|< 500|Static collisions (Option 1)|
|500 - 5,000|Proximity pooling (Option 2)|
|5,000+|Baked mesh OR skip collision|
|Grass/foliage|No collision needed|

---

## Editor Workflow with @tool

The `@tool` annotation lets scripts run in the editor—essential for placing and previewing instances.

### Basic @tool Structure

```gdscript
@tool
extends Node3D

@export var some_property: float = 1.0 : set = _set_property

func _set_property(value):
    some_property = value
    if Engine.is_editor_hint():
        _update_visuals()
```

### Export Actions Pattern

Use `bool` exports as buttons:

```gdscript
@tool
extends Node3D

@export_group("Actions")
@export var generate: bool = false : set = _generate
@export var clear: bool = false : set = _clear

func _generate(value):
    if !value: return  # Only trigger on true
    # Do generation...

func _clear(value):
    if !value: return
    # Clear children...
```

In the inspector, these appear as checkboxes. Clicking them triggers the setter.

### Editor vs Runtime

```gdscript
func _ready():
    if Engine.is_editor_hint():
        # Editor-only code
        return
    
    # Runtime code here

func _process(delta):
    if Engine.is_editor_hint():
        return  # Don't run game logic in editor
    
    # Game logic
```

### Persisting Generated Nodes

To keep generated nodes when saving the scene:

```gdscript
func _create_node():
    var node = Node3D.new()
    add_child(node)
    
    # This line makes it save with the scene
    node.owner = get_tree().edited_scene_root
```

Without setting `owner`, child nodes are lost on scene reload.

---

## Complete System Example

### Scene Structure

```
TreeSystem (Node3D)
├── Positions (Node3D)       ← Marker3D nodes go here
├── Visuals (Node3D)         ← Generated MultiMeshes
└── Collisions (Node3D)      ← Generated StaticBodies
```

### Full Script

```gdscript
@tool
extends Node3D

## The mesh to instance
@export var tree_mesh: Mesh

## Collision shape dimensions
@export var trunk_radius: float = 0.3
@export var trunk_height: float = 2.0

## Size of each chunk for culling
@export var chunk_size: float = 32.0

@export_group("Actions")
@export var generate: bool = false : set = _on_generate
@export var clear: bool = false : set = _on_clear

# ============================================================
# ACTIONS
# ============================================================

func _on_generate(value: bool) -> void:
    if !value: return
    _on_clear(true)
    
    var positions := _collect_positions()
    if positions.is_empty():
        push_warning("No positions found under Positions node")
        return
    
    var chunks := _sort_into_chunks(positions)
    _create_multimeshes(chunks)
    _create_collisions(positions)
    
    print("Generated %d trees in %d chunks" % [positions.size(), chunks.size()])

func _on_clear(value: bool) -> void:
    if !value: return
    
    for child in $Visuals.get_children():
        child.queue_free()
    for child in $Collisions.get_children():
        child.queue_free()

# ============================================================
# POSITION COLLECTION
# ============================================================

func _collect_positions() -> Array[Vector3]:
    var positions: Array[Vector3] = []
    
    for child in $Positions.get_children():
        if child is Node3D:
            positions.append(child.global_position)
    
    return positions

# ============================================================
# CHUNKING
# ============================================================

func _sort_into_chunks(positions: Array[Vector3]) -> Dictionary:
    var chunks: Dictionary = {}
    
    for pos in positions:
        var coord := Vector2i(
            floori(pos.x / chunk_size),
            floori(pos.z / chunk_size)
        )
        
        if coord not in chunks:
            chunks[coord] = []
        chunks[coord].append(pos)
    
    return chunks

# ============================================================
# MULTIMESH GENERATION
# ============================================================

func _create_multimeshes(chunks: Dictionary) -> void:
    for coord in chunks:
        var positions: Array = chunks[coord]
        
        var mm := MultiMesh.new()
        mm.mesh = tree_mesh
        mm.transform_format = MultiMesh.TRANSFORM_3D
        mm.instance_count = positions.size()
        
        for i in positions.size():
            var xform := Transform3D()
            xform.origin = positions[i]
            mm.set_instance_transform(i, xform)
        
        var mmi := MultiMeshInstance3D.new()
        mmi.multimesh = mm
        mmi.name = "Chunk_%d_%d" % [coord.x, coord.y]
        
        $Visuals.add_child(mmi)
        mmi.owner = get_tree().edited_scene_root

# ============================================================
# COLLISION GENERATION
# ============================================================

func _create_collisions(positions: Array[Vector3]) -> void:
    # Shared shape for memory efficiency
    var shape := CylinderShape3D.new()
    shape.radius = trunk_radius
    shape.height = trunk_height
    
    for i in positions.size():
        var body := StaticBody3D.new()
        body.name = "Col_%d" % i
        
        var col := CollisionShape3D.new()
        col.shape = shape
        body.add_child(col)
        
        $Collisions.add_child(body)
        body.owner = get_tree().edited_scene_root
        col.owner = get_tree().edited_scene_root
        
        # Offset up so cylinder base is at position
        body.global_position = positions[i] + Vector3(0, trunk_height / 2.0, 0)
```

---

## Placing Positions on Terrain

### Manual Placement

1. Add `Marker3D` nodes under `Positions`
2. Use Godot's snap tools + surface snapping
3. Position each where you want a tree

### Raycast Placement Tool

```gdscript
@tool
extends Node3D

@export var placement_enabled: bool = false

func _input(event: InputEvent) -> void:
    if !Engine.is_editor_hint() or !placement_enabled:
        return
    
    if event is InputEventMouseButton and event.pressed:
        match event.button_index:
            MOUSE_BUTTON_LEFT:
                _place_at_cursor(event.position)
            MOUSE_BUTTON_RIGHT:
                _remove_at_cursor(event.position)

func _place_at_cursor(screen_pos: Vector2) -> void:
    var result := _raycast_terrain(screen_pos)
    if result.is_empty():
        return
    
    var marker := Marker3D.new()
    $Positions.add_child(marker)
    marker.owner = get_tree().edited_scene_root
    marker.global_position = result.position

func _remove_at_cursor(screen_pos: Vector2) -> void:
    var result := _raycast_terrain(screen_pos)
    if result.is_empty():
        return
    
    var closest: Node3D = null
    var closest_dist := 2.0  # Max selection distance
    
    for child in $Positions.get_children():
        var dist: float = child.global_position.distance_to(result.position)
        if dist < closest_dist:
            closest_dist = dist
            closest = child
    
    if closest:
        closest.queue_free()

func _raycast_terrain(screen_pos: Vector2) -> Dictionary:
    var viewport := EditorInterface.get_editor_viewport_3d(0)
    var camera := viewport.get_camera_3d()
    
    var from := camera.project_ray_origin(screen_pos)
    var dir := camera.project_ray_normal(screen_pos)
    var to := from + dir * 1000.0
    
    var query := PhysicsRayQueryParameters3D.create(from, to)
    var space := get_world_3d().direct_space_state
    
    return space.intersect_ray(query)
```

---

## Per-Instance Variation

### Custom Data

MultiMesh supports 4 floats per instance via `INSTANCE_CUSTOM`:

```gdscript
# When setting up instances
mm.use_custom_data = true

for i in instance_count:
    # Store random values for shader
    var custom := Color(
        randf(),        # Wind phase offset
        randf(),        # Scale variation
        randf(),        # Color variation
        0.0             # Unused
    )
    mm.set_instance_custom_data(i, custom)
```

### In Shader

```gdshader
shader_type spatial;

uniform sampler2D albedo_texture : source_color;

void vertex() {
    float wind_offset = INSTANCE_CUSTOM.r;
    float scale_var = INSTANCE_CUSTOM.g;
    
    // Apply wind animation with offset
    float wind = sin(TIME + wind_offset * TAU) * 0.1;
    VERTEX.x += wind * UV.y;  // Stronger at top
    
    // Apply scale variation
    VERTEX *= mix(0.8, 1.2, scale_var);
}

void fragment() {
    float color_var = INSTANCE_CUSTOM.b;
    vec3 tex = texture(albedo_texture, UV).rgb;
    
    // Tint variation
    ALBEDO = mix(tex, tex * vec3(0.9, 1.0, 0.8), color_var);
}
```

---

## Performance Reference

### Draw Call Comparison

|Method|1,000 objects|10,000 objects|
|---|---|---|
|Individual MeshInstance3D|1,000 calls|10,000 calls|
|MultiMesh (1 chunk)|1 call|1 call|
|MultiMesh (chunked 32m)|~10-50 calls|~100-500 calls|

### Instance Limits (rough estimates)

|Hardware|Simple mesh (24 tris)|Complex mesh (500 tris)|
|---|---|---|
|Low-end|~10,000|~2,000|
|Mid-range|~60,000|~10,000|
|High-end|~200,000+|~30,000+|

These vary wildly based on shaders, shadows, and other factors.

### Memory

- Each instance: 48 bytes (transform) + 16 bytes (custom data if used)
- 10,000 instances ≈ 640 KB for transform data alone
- Collision bodies add significant memory overhead

---

## Common Issues

### Instances Not Visible

1. Check `instance_count` is set before transforms
2. Verify mesh is assigned
3. Check material/shader isn't culling

### Flickering / Z-Fighting

Instances at exact same position. Add tiny random offset:

```gdscript
xform.origin += Vector3(randf() * 0.01, 0, randf() * 0.01)
```

### Changes Not Saving

Set `owner` on generated nodes:

```gdscript
node.owner = get_tree().edited_scene_root
```

### Performance Still Bad

- Check shadow settings (biggest culprit)
- Reduce mesh complexity
- Increase chunk size if too fragmented
- Use LOD or impostors for distant instances

---

## Related Topics

- [[Triplanar Mapping]] — Texture without UVs
- [[Godot Shaders]] — Custom rendering
- [[Frustum Culling]] — How visibility culling works
- [[LOD System]] — Level of detail for distant objects

---

## Quick Reference

```gdscript
# Create MultiMesh
var mm = MultiMesh.new()
mm.mesh = mesh_resource
mm.transform_format = MultiMesh.TRANSFORM_3D
mm.instance_count = count
mm.set_instance_transform(index, transform)

# Create node
var mmi = MultiMeshInstance3D.new()
mmi.multimesh = mm

# Custom data (optional)
mm.use_custom_data = true
mm.set_instance_custom_data(index, Color(r, g, b, a))

# In shader: INSTANCE_CUSTOM.rgba
```