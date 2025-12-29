## Overview

Raycasting from camera is essential for click-to-place tools, selection systems, and any editor interaction that needs to know "what's under the cursor."

## Basic Raycast from Screen Position

```gdscript
func raycast_from_screen(camera: Camera3D, screen_pos: Vector2) -> Dictionary:
    # Convert screen position to ray
    var from = camera.project_ray_origin(screen_pos)
    var direction = camera.project_ray_normal(screen_pos)
    var to = from + direction * 1000.0  # Ray length
    
    # Get physics space
    var space = camera.get_world_3d().direct_space_state
    
    # Create and execute query
    var query = PhysicsRayQueryParameters3D.create(from, to)
    return space.intersect_ray(query)
```

## Understanding the Result

`intersect_ray()` returns a Dictionary:

```gdscript
var result = space.intersect_ray(query)

if result:
    result.position   # Vector3: Hit point in world space
    result.normal     # Vector3: Surface normal at hit point
    result.collider   # Object: The CollisionObject3D that was hit
    result.rid        # RID: Resource ID of the collider
    result.shape      # int: Shape index that was hit
else:
    # Ray hit nothing
    pass
```

## Query Parameters

```gdscript
var query = PhysicsRayQueryParameters3D.create(from, to)

# Collision mask (which layers to check)
query.collision_mask = 1  # Only layer 1

# Exclude specific objects
query.exclude = [self.get_rid(), some_other_object.get_rid()]

# Hit from inside shapes
query.hit_from_inside = true

# Collide with areas
query.collide_with_areas = true

# Collide with bodies (default true)
query.collide_with_bodies = true
```

## In EditorPlugin Context

```gdscript
func _forward_3d_gui_input(camera: Camera3D, event: InputEvent) -> int:
    if event is InputEventMouseButton:
        if event.button_index == MOUSE_BUTTON_LEFT and event.pressed:
            var result = _raycast(camera, event.position)
            if result:
                print("Hit at: ", result.position)
                print("Normal: ", result.normal)
                print("Object: ", result.collider.name)
            else:
                print("No hit")
            return AFTER_GUI_INPUT_STOP
    return AFTER_GUI_INPUT_PASS

func _raycast(camera: Camera3D, screen_pos: Vector2) -> Dictionary:
    var from = camera.project_ray_origin(screen_pos)
    var to = from + camera.project_ray_normal(screen_pos) * 1000.0
    var space = camera.get_world_3d().direct_space_state
    var query = PhysicsRayQueryParameters3D.create(from, to)
    return space.intersect_ray(query)
```

## Placing Objects at Hit Point

```gdscript
func place_at_cursor(camera: Camera3D, screen_pos: Vector2):
    var result = _raycast(camera, screen_pos)
    if not result:
        return
    
    var instance = mesh_scene.instantiate()
    get_tree().edited_scene_root.add_child(instance)
    instance.owner = get_tree().edited_scene_root
    instance.global_position = result.position
```

## Aligning to Surface Normal

```gdscript
func place_aligned_to_surface(position: Vector3, normal: Vector3):
    var instance = mesh_scene.instantiate()
    # ... add to tree ...
    
    instance.global_position = position
    
    # Align Y-up to surface normal
    if normal != Vector3.UP and normal != Vector3.DOWN:
        var basis = Basis()
        basis.y = normal
        basis.x = normal.cross(Vector3.UP).normalized()
        basis.z = basis.x.cross(normal).normalized()
        instance.global_basis = basis
    elif normal == Vector3.DOWN:
        instance.rotate_x(PI)  # Flip upside down
```

## Filtering Hits

### By Collision Layer

```gdscript
func _raycast_terrain_only(camera: Camera3D, screen_pos: Vector2) -> Dictionary:
    var from = camera.project_ray_origin(screen_pos)
    var to = from + camera.project_ray_normal(screen_pos) * 1000.0
    var space = camera.get_world_3d().direct_space_state
    var query = PhysicsRayQueryParameters3D.create(from, to)
    query.collision_mask = 2  # Only layer 2 (terrain)
    return space.intersect_ray(query)
```

### Exclude Self/Generated Objects

```gdscript
func _raycast_excluding(camera: Camera3D, screen_pos: Vector2, exclude: Array[RID]) -> Dictionary:
    var from = camera.project_ray_origin(screen_pos)
    var to = from + camera.project_ray_normal(screen_pos) * 1000.0
    var space = camera.get_world_3d().direct_space_state
    var query = PhysicsRayQueryParameters3D.create(from, to)
    query.exclude = exclude
    return space.intersect_ray(query)
```

## Multiple Hits (Raycast All)

For getting all objects along the ray, use shape casting with a thin ray-like shape, or perform multiple raycasts excluding previous hits:

```gdscript
func raycast_all(camera: Camera3D, screen_pos: Vector2, max_hits: int = 10) -> Array:
    var hits = []
    var exclude = []
    
    for i in max_hits:
        var from = camera.project_ray_origin(screen_pos)
        var to = from + camera.project_ray_normal(screen_pos) * 1000.0
        var space = camera.get_world_3d().direct_space_state
        var query = PhysicsRayQueryParameters3D.create(from, to)
        query.exclude = exclude
        
        var result = space.intersect_ray(query)
        if result:
            hits.append(result)
            exclude.append(result.rid)
        else:
            break
    
    return hits
```

## Common Issues

### Raycast Returns Empty

1. **No collision shapes**: The objects need `CollisionShape3D` nodes
2. **Wrong collision layer/mask**: Check your collision settings
3. **Ray too short**: Increase the ray length
4. **Physics not ready**: In editor, physics might need a frame to initialize

### Hitting Wrong Objects

1. **Check collision layers**: Use specific layers for terrain vs props
2. **Exclude objects**: Use `query.exclude` to skip certain objects

### Performance

Raycasting is cheap. Don't worry about doing it on every mouse move if needed. Just don't do thousands per frame.

## 2D Raycasting

Similar approach for 2D:

```gdscript
func _forward_canvas_gui_input(event: InputEvent) -> bool:
    if event is InputEventMouseButton and event.pressed:
        var space = get_viewport().world_2d.direct_space_state
        var query = PhysicsPointQueryParameters2D.new()
        query.position = event.position
        var result = space.intersect_point(query)
        # result is an Array of dictionaries
        return true
    return false
```

---
# Raycast Essentials

```gdscript
camera.project_ray_origin(screen_pos)    # Ray start point
camera.project_ray_normal(screen_pos)    # Ray direction
camera.get_world_3d().direct_space_state # Physics space
PhysicsRayQueryParameters3D.create(from, to)
space.intersect_ray(query)               # Returns Dictionary or {}
result.position                          # Hit point
result.normal                            # Surface normal
result.collider                          # Hit object
```