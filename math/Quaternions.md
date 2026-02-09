> [[Euler angles]] are the directions (turn left, look up), while quaternions are coordinates (you are here)

4 numbers describe an orientation in 4D, where each number is equally important.
- w: [[scalar]] that balances others 3 coordinates. like a slider bar.
- (x, y, z) 3D vector, which represents the axis of rotation, scaled by the rotation amount

We can think of 4 numbers as 4 "*weights*" giving the **blend amount** of certain representation.

| Quaternion (x,y,z,w)            | Meaning       | Visual Analogy                       |
| ------------------------------- | ------------- | ------------------------------------ |
| **0, 0, 0, 1**                  | Identity (0°) | Car parked normally.                 |
| **1, 0, 0, 0**                  | 180° on X     | Car flipped upside down.             |
| **0, 1, 0, 0**                  | 180° on Y     | Car facing backwards.                |
| **0, 0, 1, 0**                  | 180° on Z     | Car doing a barrel roll (roof down). |
| **1, 0, 0, 1** (not normalized) | 90° on X      | Car driving on the right wall.       |
|                                 |               |                                      |
> [!warning] Normalization
> Note that last example is not a normalized quaternions, they are just for understanding. Real quaternions are normalized, because they have "see-saw" relationship. And in real use case for example Unity will normalize before applying.
> `w` and `(x,y,z)` are on opposite sides of a see-saw.
> $$w2+(x2+y2+z2)=1$$
> - If `w` takes up all the space (`w=1`), there is **zero room left** for `x, y, z`. They _must_ be 0. (0° rotation).
> - If `w` takes up no space (`w=0`), then `x, y, z` take up **all the space**. They are at their maximum values. (180° rotation).

> We also can say (0, 0, 0, 0) which is the "zero element", but it is not a valid rotation, because having w at 0 simply scaling the object down into singularity


The point on a sphere nature of the quaternions allows us to mix them by drawing a line between two points.

---
# Unity Cheat Sheet

## 1. Creating Rotations

How to convert human ideas into Quaternions.

| Goal                 | Code                                  | Notes                                                     |
| -------------------- | ------------------------------------- | --------------------------------------------------------- |
| **Reset Rotation**   | `Quaternion.identity`                 | The "zero" rotation (0,0,0,1).                            |
| **Use Euler Angles** | `Quaternion.Euler(x, y, z)`           | Most common. Converts (0-360) angles to Quaternion.       |
| **Look at Target**   | `Quaternion.LookRotation(direction)`  | Points Z-axis (Blue) along the `direction` vector.        |
| **Angle Axis**       | `Quaternion.AngleAxis(angle, axis)`   | Rotates `angle` degrees around `axis` (e.g., Vector3.up). |
| **Align Vectors**    | `Quaternion.FromToRotation(from, to)` | Creates rotation needed to get from vector A to vector B. |

## 2. Applying Rotations

How to combine and use them.

|Goal|Code|Math Equivalence|
|---|---|---|
|**Combine Rotations**|`rotA * rotB`|Apply B, **then** apply A (Order is Right-to-Left!).|
|**Rotate a Vector**|`myRot * myVector`|Takes a direction vector and rotates it by the quaternion.|
|**Smooth Rotate**|`Quaternion.Slerp(a, b, t)`|Spherical interpolation. `t` is 0 to 1.|
|**Rotate Towards**|`Quaternion.RotateTowards(curr, target, maxDeg)`|Like Slerp, but you specify max degrees per frame (great for turrets).|
|**Inverse**|`Quaternion.Inverse(rot)`|The "Undo" button. `rot * Inverse(rot) == identity`.|

## 3. Debugging / Reading

How to see what's happening.

|Goal|Code|Notes|
|---|---|---|
|**Get Angle**|`Quaternion.Angle(a, b)`|Returns degrees difference between two rotations.|
|**To Euler**|`myRot.eulerAngles`|Converts back to Vector3 (0-360). **Read only!** Don't modify and re-assign.|
|**Check Direction**|`transform.forward`|The result of `rotation * Vector3.forward`.|

## 4. Common Patterns

**Smooth Camera Look**
```csharp
// Slerp to a new target rotation
transform.rotation = Quaternion.Slerp(transform.rotation, targetRot, Time.deltaTime * speed);`
```

**FPS Character Rotation (The Accumulator)**
```csharp
// Store angles in floats (not quaternions)
float yaw += Input.GetAxis("Mouse X"); float pitch -= Input.GetAxis("Mouse Y"); // Apply them all at once
transform.rotation = Quaternion.Euler(pitch, yaw, 0);`
```

**Turret Tracking**
```csharp
Vector3 direction = target.position - transform.position; Quaternion targetRot = Quaternion.LookRotation(direction); transform.rotation = Quaternion.RotateTowards(transform.rotation, targetRot, turnSpeed * Time.deltaTime);`
```
## The Forbidden Zone

- `myRot.x = 0.5f;` -> **NEVER** modify components directly.
- `myRot.eulerAngles.y += 10;` -> **AVOID** modifying eulerAngles property directly (drift issues).
- `rotA + rotB` -> **NONSENSE**. You multiply quaternions, you don't add them.