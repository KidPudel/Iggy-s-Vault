# Quaternions

A number system extending complex numbers that represents 3D rotations using four components: x, y, z, and w.

## What it does

- Defined as **q = (x, y, z, w)** where `(x, y, z)` is a 3D vector scaled by the sine of half the rotation angle, and `w` is the cosine of half the rotation angle
- Valid rotations are **unit quaternions** (normalized): w² + x² + y² + z² = 1
- `w` and the magnitude of `(x, y, z)` are on a see-saw: when w = 1, xyz = 0 (identity/no rotation); when w = 0, xyz is at maximum (180° rotation)
- Rotations are composed by **multiplication** (not addition): `q1 * q2` applies q2 first, then q1
- Quaternion multiplication is non-commutative: `q1 * q2 ≠ q2 * q1`
- The identity quaternion `(0, 0, 0, 1)` represents no rotation
- `(0, 0, 0, 0)` is not a valid rotation
- **Slerp** (spherical linear interpolation) interpolates smoothly between two quaternions along the shortest arc on the unit 4D sphere
- Inverse of a unit quaternion `q` is its conjugate `(-x, -y, -z, w)`; `q * q⁻¹ = identity`
- Avoid gimbal lock because they operate in 4D space with no nested axis hierarchy

## Code

```
Identity (no rotation):  (0, 0, 0, 1)
180° on X axis:          (1, 0, 0, 0)
180° on Y axis:          (0, 1, 0, 0)
180° on Z axis:          (0, 0, 1, 0)

Normalization constraint: w² + x² + y² + z² = 1

Composition (apply B then A): result = A * B
```

## Sources

- [TODO: find source]

## Related

- [[euler-angles]]
- [[gimbal-lock]]
- [[rotation-matrix]]
- [[slerp]]

## Process

- Why does quaternion multiplication represent rotation composition rather than addition?
- How does the unit-sphere constraint (normalization) ensure a quaternion represents a valid rotation?
- What property of quaternion space makes slerp produce a constant-speed interpolation path?
- How does representing rotation as half-angles in the quaternion components avoid gimbal lock?
- What is the relationship between a quaternion and the rotation matrix it represents?
