### 3D Transformations in Godot

---

**Transform Structure**

Every 3D node has two transforms:

- `transform` — local (relative to parent)
- `global_transform` — world space (all parent transforms applied)

Both contain:

- `origin` — position (Vector3)
- `basis` — rotation and scale (3x3 matrix)

---

**Basis**

A 3x3 matrix holding the node's local axes as world-space vectors:

- `basis.x` — node's right direction
- `basis.y` — node's up direction
- `basis.z` — node's forward direction

Multiply to convert directions:

```gdscript
world_direction = node.basis * local_direction
local_direction = node.basis.inverse() * world_direction
```

---

**Local vs Global**

```
Parent (rotated 45°)
└── Child (rotated 0° locally)
    └── child.transform.basis        → identity (no local rotation)
    └── child.global_transform.basis → includes parent's 45°
```

Use `global_transform` when you need actual world orientation from nested nodes.

---

**Common Operations**

|Task|Code|
|---|---|
|Get world position|`node.global_position`|
|Get facing direction|`node.global_transform.basis.z`|
|Convert local → world direction|`node.global_transform.basis * vector`|
|Face a point|`node.look_at(target_position, Vector3.UP)`|
|Rotate around axis|`node.rotate_y(angle)`|

---

**Vector Essentials**

- `.normalized()` — length = 1, prevents fast diagonals
- `.length()` — distance/magnitude
- `move_toward(current, target, delta)` — clamped interpolation
- `lerp(a, b, t)` — linear interpolation (t = 0–1)

---

**Rotation Gotcha**

Mouse/screen axes ≠ rotation axes:

- Mouse X (horizontal) → `rotate_y()` (vertical axis)
- Mouse Y (vertical) → `rotate_x()` (horizontal axis)
- Negate for intuitive controls

---

**Hierarchy Rule**

Transforms cascade. Rotate a parent → all children rotate with it. This is why SpringArm3D works: rotate the arm, camera follows without touching camera's local transform.

---

**When to Use What**

|Situation|Use|
|---|---|
|Movement relative to camera|`camera.global_transform.basis`|
|Check node's local rotation|`node.transform.basis`|
|Spawn object at world position|`global_position`|
|Attach child that shouldn't inherit rotation|Use `top_level = true` on child|

---

**Further Reading**

- Quaternions — `basis.get_rotation_quaternion()`, avoid gimbal lock
- `slerp()` — smooth rotation interpolation
- `Basis.looking_at()` — create basis facing a direction

That's the mental model. Everything else builds on this.