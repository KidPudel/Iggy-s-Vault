# Basis in Godot

## Definition

`Basis` is a 3x3 matrix that stores an object's local axes in 3D:
- `basis.x` right
- `basis.y` up
- `basis.z` backward (forward is `-basis.z`)

Axis direction encodes rotation. Axis length encodes scale.

## Common use

Convert local direction to world direction:

```gdscript
var world_dir: Vector3 = node.global_transform.basis * local_dir
```

Convert world direction to local direction:

```gdscript
var local_dir: Vector3 = node.global_transform.basis.inverse() * world_dir
```

## Why it matters

- Character movement is often local intent that must become world-space velocity.
- Camera-relative movement depends on basis conversion.

## See also

- [[Transform3D in Godot]]
- [[matrix multiplication]]
- [[Quaternions]]
