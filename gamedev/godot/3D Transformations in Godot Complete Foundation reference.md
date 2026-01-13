## Transform Structure

Every 3D node has two transforms:
- transform — local (relative to parent)
- global_transform — world space (all parent transforms applied)

Both contain:
- origin — position (Vector3)
- basis — rotation and scale (3x3 matrix)

The origin is the node's pivot point/anchor in 3D space. By default, it's at 
the geometric center of meshes, but can be adjusted in modeling software.

---

## Coordinate System

Godot uses Y-up, right-handed coordinates:
- +X → right
- +Y → up  
- +Z → backward (toward camera in editor)
- -Z → forward (away from camera)

This affects all direction vectors and rotations.

---

## Basis (The 3x3 Matrix)

A 3x3 matrix holding the node's local axes as world-space vectors:
- basis.x — node's right direction
- basis.y — node's up direction  
- basis.z — node's forward direction (negative for "ahead")

Each axis vector's:
- Direction = rotation
- Length = scale on that axis

Identity basis (no transform):
  Basis(Vector3(1,0,0), Vector3(0,1,0), Vector3(0,0,1))

Multiply to convert directions:
  world_direction = node.basis * local_direction
  local_direction = node.basis.inverse() * world_direction

Why this matters: Input vectors are usually in local space (WASD), but 
velocity needs world space. The basis converts between them.

---

## Local vs Global

Example hierarchy:
```
  Parent (rotated 45°)
  └── Child (rotated 0° locally)
      └── child.transform.basis        → identity (no local rotation)
      └── child.global_transform.basis → includes parent's 45°
```

Use global_transform when you need actual world orientation from nested nodes.

Rule: Transforms cascade down. Rotate parent → all children rotate with it.
This is why SpringArm3D works: rotate the arm, camera follows without 
touching camera's local transform.

---

## Common Operations

| Task                       | Code                                           |
| -------------------------- | ---------------------------------------------- |
| Get world position         | node.global_position                           |
| Get facing direction       | node.global_transform.basis.z                  |
| Convert local → world dir  | node.global_transform.basis * vector           |
| Convert world → local dir  | node.global_transform.basis.inverse() * vector |
| Face a point               | node.look_at(target, Vector3.UP)               |
| Rotate around local axis   | node.rotate_y(angle)                           |
| Rotate around world axis   | node.rotate(Vector3.UP, angle)                 |
| Move forward in facing dir | node.translate(-transform.basis.z * speed)     |

---

## Vector Essentials

- .normalized() — length = 1, prevents fast diagonals in movement
- .length() — distance/magnitude from origin
- .distance_to(other) — distance between two points
- .dot(other) — similarity of direction (-1 to 1)
- .cross(other) — perpendicular vector (right-hand rule)
- move_toward(current, target, delta) — clamped interpolation
- lerp(a, b, t) — linear interpolation (t = 0–1)

---

## Rotation Methods

Three representations:
1. Basis — 3x3 matrix (what's stored in transform)
2. Euler angles — 3 rotation values (degrees/radians)
3. Quaternion — 4D rotation (no gimbal lock)

Convert between them:
  var euler = basis.get_euler()
  var quat = basis.get_rotation_quaternion()
  var basis = Basis.from_euler(euler)

Use quaternions for smooth rotation interpolation:
  var smooth_rot = quat_a.slerp(quat_b, weight)

---

## Rotation Gotcha

Mouse/screen axes ≠ rotation axes:
- Mouse X (horizontal) → rotate_y() (vertical axis)
- Mouse Y (vertical) → rotate_x() (horizontal axis)  
- Negate mouse delta for intuitive FPS controls

This is because you're rotating AROUND axes, not IN the axis direction.

---

## Transform Operations Order

Order matters when combining transforms:
  transform = parent_transform * local_transform

This is why:
  rotate(a) then translate(b) ≠ translate(b) then rotate(a)

For complex movements, accumulate rotations separately from translation.

---

## When to Use What

| Situation | Use |
|-----------|-----|
| Movement relative to camera | camera.global_transform.basis |
| Check node's local rotation | node.transform.basis |
| Spawn object at world position | global_position |
| Attach child that ignores parent rot | top_level = true on child |
| Character movement with input | character.basis * input_vector |
| Check if looking at target | -basis.z.dot(direction_to_target) |

---

## Physics Body Specifics

CharacterBody3D and RigidBody3D handle transforms differently:
- CharacterBody3D.velocity — always in world space
- Use move_and_slide() for collision-aware movement
- Don't modify transform directly during physics processing
- velocity = transform.basis * local_velocity converts properly

---

## Related Concepts
- [[Quaternions]] — basis.get_rotation_quaternion(), smooth interpolation
- [[Gimbal Lock]] — why Euler angles fail at 90° rotations  
- [[Matrix Multiplication]] — order matters, not commutative
- [[Local vs Global Space]] — parent-relative vs world-absolute
- [[Homogeneous Coordinates]] — why Transform3D is 3x4 not 3x3

---

## Common Mistakes

1. Forgetting to normalize direction vectors → inconsistent speeds
2. Using transform when you need global_transform in nested scenes
3. Rotating then translating when you meant the opposite order
4. Comparing floats with == instead of is_equal_approx()
5. Mixing local/global operations without converting properly

---

## Further Reading
- Basis.looking_at() — create basis facing a direction
- Transform3D.interpolate_with() — blend between transforms
- Vector3.FORWARD/RIGHT/UP constants — semantic direction names

That's the complete foundation. Everything else builds on this.
