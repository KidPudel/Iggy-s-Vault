# Euler Angles

A representation of 3D orientation using three sequential rotation angles around a fixed set of axes.

## What it does

- Three angles — **Yaw**, **Pitch**, and **Roll** — each rotate around one of the object's local axes
- Yaw rotates around the up axis, pitch around the left/right axis, roll around the forward axis
- Rotations are applied in a fixed order (e.g. Yaw → Pitch → Roll), forming a hierarchy of nested rings
- Because axes are local and nested, the order of application is non-commutative — changing the order changes the result
- To represent orientation with only 3 independent values, the axes are linked in a hierarchy rather than kept fully local
- This nesting produces **gimbal lock**: when two nested rings align, one degree of freedom is lost and two axes produce the same rotation
- Gimbal lock occurs when a middle rotation (e.g. pitch of 90°) causes the outer and inner axes to coincide

## Code

```
Rotation order: Yaw → Pitch → Roll
Apply Pitch 90°, then Roll 90°  ≠  Apply Roll 90°, then Pitch 90°
```

## Sources

- [TODO: find source]

## Related

- [[quaternions]]
- [[rotation-matrix]]
- [[gimbal-lock]]

## Process

- Why does linking the axes in a hierarchy (rather than keeping them fully local) cause gimbal lock?
- How does the non-commutative property of Euler angle composition relate to matrix multiplication order?
- What would be needed to represent orientation without the gimbal lock limitation of Euler angles?
- How does the choice of rotation order (e.g. XYZ vs ZYX) affect which configuration triggers gimbal lock?
