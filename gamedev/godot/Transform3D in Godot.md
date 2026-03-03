# Transform3D in Godot

## Definition

`Transform3D` is a 3D transform composed of:
- `origin` (position)
- `basis` (rotation + scale)

It represents where an object is and how it is oriented/scaled in 3D space.

## Local vs global

- `transform`: local, relative to parent.
- `global_transform`: world space, after parent transforms are applied.

## Why it matters

- Movement and orientation bugs often come from mixing local and global transforms.
- Physics and camera logic often need world-space values.

## See also

- [[Basis in Godot]]
- [[Local vs global transform in Godot]]
- [[3D Transformations in Godot Complete Foundation reference]]
