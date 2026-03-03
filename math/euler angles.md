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

---
<!-- original content preserved below -->
# Euler Angles (original)

3 angles describe orientation in 3d world.

> here is the axis agnostic version, since everyone uses whatever the hell they want apparently

1. Yaw - rotates around UP-axis (looks like looking sides without moving the eyes)
2. Pitch - rotates around LEFT-axis (tilting forward and back)
3. Roll - rotates around FORWARD-axis (tilting on the sides)

These axes are object's pivots.

Now, try the following test:
> [!note] TEST
> Grab your phone or book and try two following approaches remembering about their initial directions/pivot sticks
> 1. **Sequence A:** Rotate it 90° toward you (Pitch), then rotate it 90° clockwise (Roll).
> 2. **Sequence B:** Start over. Rotate it 90° clockwise (Roll) first, then 90° toward you (Pitch).

From this test we can understand that the ***order matters*** (non-commutative). And it matters, because the pivots of the object are local.
This is conveniently represented in this image.

But since it is not just a free ball rotation without any responsibilities, but instead it is used for computation therefore recreation, we have some limitations.
In order to make it easily recreated just using 3 numbers, we must *link* them together making them 3 independent knobs.

> Otherwise if they were all truly local we would need to use Rotation Matrix (which uses 9 numbers).

And we link them by nesting them in a hierarchy, that we can visualize as three nested rings in the following order: Yaw -> Pitch -> Roll

And since the Yaw is the outer ring, when we pitch by 90, the Yaw won't change its position and still point up, together with roll, resulting in... ***Gimbal lock***.

# Gimbal lock

The loss of one degree of freedom in 3D space, causing two rotations to do the same visible thing.

---
# Unity API Cheat Sheet

> [!warning] The Read/Write Trap
> Unity stores rotations internally as Quaternions. `eulerAngles` is a converted representation, not the source of truth.

## Properties
**`transform.eulerAngles`** (Vector3)
- **Writing:** Safe. You can assign `new Vector3(0, -90, 0)`. Unity converts this to its internal Quaternion.
- **Reading:** **Dangerous.** It recalculates the best guess 0-360 representation.
    - *Gotcha:* If you set `-10`, it reads back `350`. If you set `370`, it reads back `10`.
    - *Consequence:* Do not decrement this value directly (e.g., `x -= 1`) for accumulation, or it will snap/jitter near 0/360.

**`transform.localEulerAngles`**
- Same as above, but relative to the parent.

## Methods
**`Quaternion.Euler(x, y, z)`**
- Creates a rotation from degrees. Use this to assign to `transform.rotation`.

**`transform.Rotate(x, y, z, Space.Self)`**
- Applies rotation incrementally.
- Defaults to `Space.Self` (local axes). Use `Space.World` to rotate around global axes.

**`Mathf.DeltaAngle(current, target)`**
- Calculates the shortest difference between two angles.
- Handles the 360 wrap-around math for you (e.g., difference between 350° and 10° returns 20°, not -340°).

---

Want commutative (aka orders doesn't matter) and without gimbal locking? [[Quaternions]] is the solution.