# Local vs global transform in Godot

## Definition

- Local transform (`transform`) is relative to the parent node.
- Global transform (`global_transform`) is absolute in world space.

## Example

If a parent rotates 45 degrees:
- Child local rotation can stay identity.
- Child global rotation includes that parent 45-degree rotation.

## Practical rule

- Use `transform` for parent-relative operations.
- Use `global_transform` for world-space movement, aiming, and spawning.

## Why it matters

Most transform bugs come from applying local data where global was needed (or the reverse).

## See also

- [[Transform3D in Godot]]
- [[Basis in Godot]]
- [[3D Transformations in Godot Complete Foundation reference]]
