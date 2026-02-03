# Godot Math to Unity C# Reference Guide

Comprehensive reference for developers transitioning from Godot (GDScript) to Unity (C#).

## Basic Math Functions

| Operation | Godot (GDScript) | Unity (C#) |
|-----------|------------------|------------|
| Absolute value | `abs(x)` | `Mathf.Abs(x)` |
| Ceiling | `ceil(x)` | `Mathf.Ceil(x)` |
| Floor | `floor(x)` | `Mathf.Floor(x)` |
| Round | `round(x)` | `Mathf.Round(x)` |
| Square root | `sqrt(x)` | `Mathf.Sqrt(x)` |
| Power | `pow(x, y)` | `Mathf.Pow(x, y)` |
| Sign | `sign(x)` | `Mathf.Sign(x)` |
| Min | `min(a, b)` | `Mathf.Min(a, b)` |
| Max | `max(a, b)` | `Mathf.Max(a, b)` |
| Clamp | `clamp(value, min, max)` | `Mathf.Clamp(value, min, max)` |
| Wrap | `wrap(value, min, max)` | Custom function needed |
| Ping pong | `pingpong(value, length)` | `Mathf.PingPong(value, length)` |

## Trigonometric Functions

All functions work in **radians** in both engines.

| Operation | Godot (GDScript) | Unity (C#) |
|-----------|------------------|------------|
| Sine | `sin(x)` | `Mathf.Sin(x)` |
| Cosine | `cos(x)` | `Mathf.Cos(x)` |
| Tangent | `tan(x)` | `Mathf.Tan(x)` |
| Arc sine | `asin(x)` | `Mathf.Asin(x)` |
| Arc cosine | `acos(x)` | `Mathf.Acos(x)` |
| Arc tangent | `atan(x)` | `Mathf.Atan(x)` |
| Arc tangent 2 | `atan2(y, x)` | `Mathf.Atan2(y, x)` |
| Hyperbolic sine | `sinh(x)` | `Mathf.Sinh(x)` (via Math.Sinh) |
| Hyperbolic cosine | `cosh(x)` | `Mathf.Cosh(x)` (via Math.Cosh) |
| Hyperbolic tangent | `tanh(x)` | `Mathf.Tanh(x)` (via Math.Tanh) |
****
## Mathematical Constants

| Constant | Godot (GDScript) | Unity (C#) |
|----------|------------------|------------|
| Pi (π) | `PI` | `Mathf.PI` |
| Tau (2π) | `TAU` | `Mathf.PI * 2` |
| Euler's number | `exp(1)` | `Mathf.Exp(1)` |
| Infinity | `INF` | `Mathf.Infinity` |
| Not a Number | `NAN` | `float.NaN` |

## Angle Conversion

| Operation | Godot (GDScript) | Unity (C#) |
|-----------|------------------|------------|
| Degrees to radians | `deg_to_rad(degrees)` | `degrees * Mathf.Deg2Rad` |
| Radians to degrees | `rad_to_deg(radians)` | `radians * Mathf.Rad2Deg` |

## Interpolation & Easing Functions

| Operation | Godot (GDScript) | Unity (C#) |
|-----------|------------------|------------|
| Linear interpolation | `lerp(a, b, t)` | `Mathf.Lerp(a, b, t)` |
| Lerp (unclamped) | `lerpf(a, b, t)` | `Mathf.LerpUnclamped(a, b, t)` |
| Inverse lerp | `inverse_lerp(a, b, v)` | `Mathf.InverseLerp(a, b, v)` |
| Angle lerp | `lerp_angle(a, b, t)` | `Mathf.LerpAngle(a, b, t)` |
| Smooth step | `smoothstep(from, to, x)` | `Mathf.SmoothStep(from, to, x)` |
| Move toward | `move_toward(from, to, delta)` | `Mathf.MoveTowards(from, to, delta)` |
| Exponential easing | `ease(x, curve)` | Custom or animation curves |
| Cubic bezier | `bezier_interpolate(...)` | Custom implementation |
| Cubic interpolation | `cubic_interpolate(...)` | Custom implementation |

## Modulo and Rounding

| Operation | Godot (GDScript) | Unity (C#) |
|-----------|------------------|------------|
| Modulo | `fmod(a, b)` | `a % b` or `Mathf.Repeat(t, length)` |
| Positive modulo | `posmod(a, b)` | `((a % b) + b) % b` |
| Wrap | `wrap(value, min, max)` | Custom or use `Mathf.Repeat()` |
| Snap/Step | `snapped(value, step)` | `Mathf.Round(value / step) * step` |
| Nearest power of 2 | `nearest_po2(value)` | `Mathf.NextPowerOfTwo(value)` |

## Vector2 Operations

| Operation | Godot (GDScript) | Unity (C#) |
|-----------|------------------|------------|
| Create vector | `Vector2(x, y)` | `new Vector2(x, y)` |
| Zero vector | `Vector2.ZERO` | `Vector2.zero` |
| One vector | `Vector2.ONE` | `Vector2.one` |
| Up vector | `Vector2.UP` | `Vector2.up` |
| Down vector | `Vector2.DOWN` | `Vector2.down` |
| Left vector | `Vector2.LEFT` | `Vector2.left` |
| Right vector | `Vector2.RIGHT` | `Vector2.right` |
| Length | `vector.length()` | `vector.magnitude` |
| Length squared | `vector.length_squared()` | `vector.sqrMagnitude` |
| Normalize | `vector.normalized()` | `vector.normalized` |
| Distance | `vector.distance_to(other)` | `Vector2.Distance(vector, other)` |
| Dot product | `vector.dot(other)` | `Vector2.Dot(vector, other)` |
| Cross product | `vector.cross(other)` | N/A (2D cross returns scalar) |
| Angle to | `vector.angle_to(other)` | `Vector2.Angle(vector, other)` |
| Angle to point | `vector.angle_to_point(other)` | `Mathf.Atan2(other.y - vector.y, other.x - vector.x)` |
| Reflect | `vector.bounce(normal)` | `Vector2.Reflect(vector, normal)` |
| Slide | `vector.slide(normal)` | `Vector2.ProjectOnPlane equivalent` |
| Lerp | `vector.lerp(to, t)` | `Vector2.Lerp(vector, to, t)` |
| Rotate | `vector.rotated(angle)` | Multiply by rotation matrix |
| Direction to | `vector.direction_to(other)` | `(other - vector).normalized` |
| Aspect ratio | `vector.aspect()` | `vector.x / vector.y` |

## Vector3 Operations

| Operation      | Godot (GDScript)             | Unity (C#)                               |
| -------------- | ---------------------------- | ---------------------------------------- |
| Create vector  | `Vector3(x, y, z)`           | `new Vector3(x, y, z)`                   |
| Zero vector    | `Vector3.ZERO`               | `Vector3.zero`                           |
| One vector     | `Vector3.ONE`                | `Vector3.one`                            |
| Up vector      | `Vector3.UP`                 | `Vector3.up`                             |
| Down vector    | `Vector3.DOWN`               | `Vector3.down`                           |
| Left vector    | `Vector3.LEFT`               | `Vector3.left`                           |
| Right vector   | `Vector3.RIGHT`              | `Vector3.right`                          |
| Forward vector | `Vector3.FORWARD`            | `Vector3.forward`                        |
| Back vector    | `Vector3.BACK`               | `Vector3.back`                           |
| Length         | `vector.length()`            | `vector.magnitude`                       |
| Length squared | `vector.length_squared()`    | `vector.sqrMagnitude`                    |
| Normalize      | `vector.normalized()`        | `vector.normalized`                      |
| Distance       | `vector.distance_to(other)`  | `Vector3.Distance(vector, other)`        |
| Dot product    | `vector.dot(other)`          | `Vector3.Dot(vector, other)`             |
| Cross product  | `vector.cross(other)`        | `Vector3.Cross(vector, other)`           |
| Reflect        | `vector.bounce(normal)`      | `Vector3.Reflect(vector, normal)`        |
| Slide          | `vector.slide(normal)`       | `Vector3.ProjectOnPlane(vector, normal)` |
| Project        | `vector.project(onto)`       | `Vector3.Project(vector, onto)`          |
| Lerp           | `vector.lerp(to, t)`         | `Vector3.Lerp(vector, to, t)`            |
| Direction to   | `vector.direction_to(other)` | `(other - vector).normalized`            |

## Basis (3x3 Matrix)

Basis represents rotation, scale, and shear as a 3x3 matrix in Godot. Unity uses Transform component and Matrix4x4 differently.

| Operation | Godot (GDScript) | Unity (C#) |
|-----------|------------------|------------|
| Identity basis | `Basis.IDENTITY` | `Matrix4x4.identity` or `Quaternion.identity` |
| From Euler angles | `Basis.from_euler(euler)` | `Matrix4x4.Rotate(Quaternion.Euler(...))` |
| From scale | `Basis.from_scale(scale)` | `Matrix4x4.Scale(scale)` |
| From axis-angle | `Basis(axis, angle)` | `Matrix4x4.Rotate(Quaternion.AngleAxis(...))` |
| Looking at | `Basis.looking_at(target, up)` | `Matrix4x4.LookAt(...)` |
| Get Euler angles | `basis.get_euler()` | `matrix.rotation.eulerAngles` |
| Get scale | `basis.get_scale()` | `matrix.lossyScale` |
| Get rotation quaternion | `basis.get_rotation_quaternion()` | `matrix.rotation` |
| Rotated | `basis.rotated(axis, angle)` | Multiply with rotation matrix |
| Scaled | `basis.scaled(scale)` | `Matrix4x4.Scale(...)` multiplication |
| Orthonormalized | `basis.orthonormalized()` | Custom or use Transform |
| Determinant | `basis.determinant()` | `matrix.determinant` |
| Inverse | `basis.inverse()` | `matrix.inverse` |
| Transposed | `basis.transposed()` | `matrix.transpose` |
| Slerp | `basis.slerp(to, weight)` | Quaternion slerp, then to matrix |
| X axis | `basis.x` | `matrix.GetColumn(0)` |
| Y axis | `basis.y` | `matrix.GetColumn(1)` |
| Z axis | `basis.z` | `matrix.GetColumn(2)` |

## Transform2D (2D Transform Matrix)

| Operation | Godot (GDScript) | Unity (C#) |
|-----------|------------------|------------|
| Create | `Transform2D(x, y, origin)` | Use Transform component (2D mode) |
| Identity | `Transform2D.IDENTITY` | Default Transform state |
| Get origin | `transform.origin` | `transform.position` |
| Set origin | `transform.origin = pos` | `transform.position = pos` |
| Get rotation | `transform.get_rotation()` | `transform.eulerAngles.z` (2D) |
| Set rotation | `transform = transform.rotated(angle)` | `transform.rotation = Quaternion.Euler(0, 0, angle)` |
| Get scale | `transform.get_scale()` | `transform.localScale` |
| Rotated | `transform.rotated(angle)` | `transform.Rotate(0, 0, angle)` |
| Scaled | `transform.scaled(scale)` | `transform.localScale *= scale` |
| Translated | `transform.translated(offset)` | `transform.Translate(offset)` |
| Inverse | `transform.affine_inverse()` | `matrix.inverse` |
| Transform point | `transform * point` | `transform.TransformPoint(point)` |
| Transform direction | `transform.basis_xform(v)` | `transform.TransformDirection(v)` |
| Inverse transform point | `transform.affine_inverse() * point` | `transform.InverseTransformPoint(point)` |

## Transform3D (3D Transform Matrix)

Godot uses a 3x4 matrix (Basis + origin). Unity uses Transform component with separate position, rotation, and scale.

| Operation                                                                                           | Godot (GDScript)                            | Unity (C#)                                  |
| --------------------------------------------------------------------------------------------------- | ------------------------------------------- | ------------------------------------------- |
| Create                                                                                              | `Transform3D(basis, origin)`                | Use Transform component                     |
| Identity                                                                                            | `Transform3D.IDENTITY`                      | Default Transform state                     |
| Get basis                                                                                           | `transform.basis`                           | Derive from rotation/scale                  |
| Get origin                                                                                          | `transform.origin`                          | `transform.position`                        |
| Set origin                                                                                          | `transform.origin = pos`                    | `transform.position = pos`                  |
| Get rotation                                                                                        | `transform.basis.get_rotation_quaternion()` | `transform.rotation`                        |
| Get Euler                                                                                           | `transform.basis.get_euler()`               | `transform.eulerAngles`                     |
| Get scale                                                                                           | `transform.basis.get_scale()`               | `transform.localScale`                      |
| Looking at                                                                                          | `transform.looking_at(target, up)`          | `transform.LookAt(target, up)`              |
| Rotated                                                                                             | `transform.rotated(axis, angle)`            | `transform.Rotate(axis, angle)`             |
| Rotated local                                                                                       | `transform.rotated_local(axis, angle)`      | `transform.Rotate(axis, angle, Space.Self)` |
| Scaled                                                                                              | `transform.scaled(scale)`                   | `transform.localScale *= scale`             |
| Scaled local                                                                                        | `transform.scaled_local(scale)`             | `transform.localScale *= scale`             |
| Translated                                                                                          | `transform.translated(offset)`              | `transform.Translate(offset, Space.World)`  |
| Translated local                                                                                    | `transform.translated_local(offset)`        | `transform.Translate(offset, Space.Self)`   |
| Inverse                                                                                             | `transform.affine_inverse()`                | `matrix.inverse`                            |
| Orthonormalized                                                                                     | `transform.orthonormalized()`               | Custom implementation                       |
| Transform point                                                                                     | `transform * point`                         | `transform.TransformPoint(point)`           |
| Transform ***local direction*** to the world space relative on from which we are looking (applying) | `transform.basis * direction`               | `transform.TransformDirection(direction)`   |
| Inverse transform point                                                                             | `transform.affine_inverse() * point`        | `transform.InverseTransformPoint(point)`    |

## Rotation & Quaternions

| Operation | Godot (GDScript) | Unity (C#) |
|-----------|------------------|------------|
| Create from Euler | `Quaternion.from_euler(euler)` | `Quaternion.Euler(x, y, z)` |
| Create from axis-angle | `Quaternion(axis, angle)` | `Quaternion.AngleAxis(angle, axis)` |
| Identity | `Quaternion.IDENTITY` | `Quaternion.identity` |
| Get euler angles | `quat.get_euler()` | `quat.eulerAngles` |
| Get axis | `quat.get_axis()` | `quat * Vector3.forward` |
| Get angle | `quat.get_angle()` | `Quaternion.Angle(identity, quat)` |
| Slerp | `quat.slerp(to, t)` | `Quaternion.Slerp(quat, to, t)` |
| Inverse | `quat.inverse()` | `Quaternion.Inverse(quat)` |
| Rotate vector | `quat * vector` | `quat * vector` |
| Look rotation | `Basis.looking_at(target, up)` | `Quaternion.LookRotation(forward, up)` |
| From basis | `Quaternion(basis)` | `matrix.rotation` |
| To basis | `Basis(quat)` | `Matrix4x4.Rotate(quat)` |

## Rect2 (2D Rectangle)

| Operation | Godot (GDScript) | Unity (C#) |
|-----------|------------------|------------|
| Create | `Rect2(position, size)` | `new Rect(x, y, width, height)` |
| Position | `rect.position` | `new Vector2(rect.x, rect.y)` |
| Size | `rect.size` | `new Vector2(rect.width, rect.height)` |
| End | `rect.end` | `new Vector2(rect.xMax, rect.yMax)` |
| Center | `rect.get_center()` | `rect.center` |
| Area | `rect.get_area()` | `rect.width * rect.height` |
| Contains point | `rect.has_point(point)` | `rect.Contains(point)` |
| Intersects | `rect.intersects(other)` | `rect.Overlaps(other)` |
| Encloses | `rect.encloses(other)` | `rect.Contains(other)` |
| Intersection | `rect.intersection(other)` | Custom implementation |
| Merge | `rect.merge(other)` | Custom or use Encapsulate |
| Grow | `rect.grow(amount)` | `rect.Expand(amount * 2)` |
| Abs | `rect.abs()` | Custom for negative sizes |

## AABB (3D Axis-Aligned Bounding Box)

| Operation | Godot (GDScript) | Unity (C#) |
|-----------|------------------|------------|
| Create | `AABB(position, size)` | `new Bounds(center, size)` |
| Position | `aabb.position` | `bounds.min` |
| Size | `aabb.size` | `bounds.size` |
| End | `aabb.end` | `bounds.max` |
| Center | `aabb.get_center()` | `bounds.center` |
| Volume | `aabb.get_volume()` | `bounds.size.x * bounds.size.y * bounds.size.z` |
| Contains point | `aabb.has_point(point)` | `bounds.Contains(point)` |
| Intersects | `aabb.intersects(other)` | `bounds.Intersects(other)` |
| Encloses | `aabb.encloses(other)` | Custom check with min/max |
| Intersection | `aabb.intersection(other)` | Custom implementation |
| Merge | `aabb.merge(other)` | `bounds.Encapsulate(other)` |
| Grow | `aabb.grow(amount)` | `bounds.Expand(amount * 2)` |
| Abs | `aabb.abs()` | Custom for negative sizes |

## Plane

| Operation | Godot (GDScript) | Unity (C#) |
|-----------|------------------|------------|
| Create | `Plane(normal, d)` | `new Plane(normal, distance)` |
| From points | `Plane(a, b, c)` | `new Plane(a, b, c)` |
| Normal | `plane.normal` | `plane.normal` |
| Distance | `plane.d` | `plane.distance` |
| Distance to point | `plane.distance_to(point)` | `plane.GetDistanceToPoint(point)` |
| Is point over | `plane.is_point_over(point)` | `plane.GetSide(point)` |
| Project point | `plane.project(point)` | `plane.ClosestPointOnPlane(point)` |
| Intersect 3 | `plane.intersect_3(b, c)` | Custom implementation |
| Intersect ray | `plane.intersects_ray(from, dir)` | `plane.Raycast(ray, out distance)` |
| Normalized | `plane.normalized()` | Custom normalization |

## Color

| Operation | Godot (GDScript) | Unity (C#) |
|-----------|------------------|------------|
| Create RGBA | `Color(r, g, b, a)` | `new Color(r, g, b, a)` |
| Create RGB | `Color(r, g, b)` | `new Color(r, g, b)` |
| White | `Color.WHITE` | `Color.white` |
| Black | `Color.BLACK` | `Color.black` |
| Red | `Color.RED` | `Color.red` |
| Green | `Color.GREEN` | `Color.green` |
| Blue | `Color.BLUE` | `Color.blue` |
| Transparent | `Color.TRANSPARENT` | `Color.clear` |
| From HTML | `Color.html(code)` | `ColorUtility.TryParseHtmlString(code, out color)` |
| To HTML | `color.to_html()` | `ColorUtility.ToHtmlStringRGBA(color)` |
| Lerp | `color.lerp(to, t)` | `Color.Lerp(color, to, t)` |
| Darkened | `color.darkened(amount)` | `color * (1 - amount)` |
| Lightened | `color.lightened(amount)` | `color + Color.white * amount` |
| Inverted | `color.inverted()` | `new Color(1-r, 1-g, 1-b, a)` |
| Luminance | `color.get_luminance()` | `color.grayscale` |

## Random Numbers

| Operation | Godot (GDScript) | Unity (C#) |
|-----------|------------------|------------|
| Random float [0,1) | `randf()` | `Random.value` |
| Random int range | `randi_range(from, to)` | `Random.Range(from, to+1)` |
| Random float range | `randf_range(from, to)` | `Random.Range(from, to)` |
| Random normal | `randfn(mean, deviation)` | Custom implementation |
| Set seed | `seed(value)` | `Random.InitState(seed)` |
| Random inside unit circle | Custom | `Random.insideUnitCircle` |
| Random inside unit sphere | Custom | `Random.insideUnitSphere` |
| Random on unit sphere | Custom | `Random.onUnitSphere` |
| Random rotation | Custom | `Random.rotation` |
| Random rotation uniform | Custom | `Random.rotationUniform` |

## Logarithms and Exponentials

| Operation | Godot (GDScript) | Unity (C#) |
|-----------|------------------|------------|
| Natural log | `log(x)` | `Mathf.Log(x)` |
| Base 10 log | `log(x) / log(10)` | `Mathf.Log10(x)` |
| Custom base log | `log(x) / log(base)` | `Mathf.Log(x, base)` |
| Exponential | `exp(x)` | `Mathf.Exp(x)` |

## Bit Operations

| Operation | Godot (GDScript) | Unity (C#) |
|-----------|------------------|------------|
| Bitwise AND | `a & b` | `a & b` |
| Bitwise OR | `a | b` | `a | b` |
| Bitwise XOR | `a ^ b` | `a ^ b` |
| Bitwise NOT | ```a` | ```a` |
| Left shift | `a << b` | `a << b` |
| Right shift | `a >> b` | `a >> b` |

## Important Differences

### Coordinate Systems
- **Godot**: Y-up for 3D (right-handed), Y-down for 2D
- **Unity**: Y-up for both 2D and 3D (left-handed for 3D)

### Forward Direction
- **Godot**: -Z is forward by default
- **Unity**: +Z is forward by default

### Rotation Order
- **Godot**: Default euler order is YXZ
- **Unity**: Default euler order is ZXY

### Transform Representation
- **Godot**: Uses Basis (3x3) + origin (Transform3D = 3x4 matrix)
- **Unity**: Uses separate position, rotation (quaternion), and scale

### Method vs Property
- **Godot**: Methods like `.length()`, `.normalized()`
- **Unity**: Properties like `.magnitude`, `.normalized`

### Global vs Instance Methods
- **Godot**: Mix of global (`lerp()`) and instance (`vector.lerp()`)
- **Unity**: Mostly static (`Mathf.Lerp()`, `Vector3.Lerp()`)

### Matrix Storage
- **Godot**: Column-major (like OpenGL)
- **Unity**: Row-major internally (like DirectX)

### Angle Units
- Both use **radians** internally
- Both provide degree/radian conversion constants

## Performance Notes

- Unity's C# benefits from IL2CPP compilation for better performance
- Godot's GDScript is dynamically typed; consider using typed hints
- Unity often requires explicit casts between int and float
- Godot's Basis operations may be faster than Unity's Matrix4x4
- Unity's Transform component caches values efficiently

## Common Patterns

### Getting Transform Axes (Godot)
```gdscript
var forward = -transform.basis.z
var right = transform.basis.x
var up = transform.basis.y
```

### Getting Transform Axes (Unity)
```csharp
var forward = transform.forward;
var right = transform.right;
var up = transform.up;
```

### Rotating to Look At (Godot)
```gdscript
transform = transform.looking_at(target_position, Vector3.UP)
```

### Rotating to Look At (Unity)
```csharp
transform.LookAt(targetPosition, Vector3.up);
```

### Local vs Global Transforms (Godot)
```gdscript
# Global
transform = transform.rotated(Vector3.UP, angle)
# Local
transform = transform.rotated_local(Vector3.UP, angle)
```

### Local vs Global Transforms (Unity)
```csharp
// Global
transform.Rotate(Vector3.up, angle, Space.World);
// Local
transform.Rotate(Vector3.up, angle, Space.Self);
```
