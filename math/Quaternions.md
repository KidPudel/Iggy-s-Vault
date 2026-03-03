 [[Euler angles]] are the directions (turn left, look up), while quaternions are coordinates (you are here)

4 numbers describe an orientation in 4D, where each number is equally important.

- w: [[scalar]] that balances others 3 coordinates. like a slider bar.
- (x, y, z) 3D vector, which represents the axis of rotation, scaled by the rotation amount

We can think of 4 numbers as 4 "_weights_" giving the **blend amount** of certain representation.

|Quaternion (x,y,z,w)|Meaning|Visual Analogy|
|---|---|---|
|**0, 0, 0, 1**|Identity (0°)|Car parked normally.|
|**1, 0, 0, 0**|180° on X|Car flipped upside down.|
|**0, 1, 0, 0**|180° on Y|Car facing backwards.|
|**0, 0, 1, 0**|180° on Z|Car doing a barrel roll (roof down).|
|**1, 0, 0, 1** (not normalized)|90° on X|Car driving on the right wall.|
||||

> [!warning] Normalization Note that last example is not a normalized quaternions, they are just for understanding. Real quaternions are normalized, because they have "see-saw" relationship. And in real use case for example Unity will normalize before applying. `w` and `(x,y,z)` are on opposite sides of a see-saw. $$w2+(x2+y2+z2)=1$$
> 
> - If `w` takes up all the space (`w=1`), there is **zero room left** for `x, y, z`. They _must_ be 0. (0° rotation).
> - If `w` takes up no space (`w=0`), then `x, y, z` take up **all the space**. They are at their maximum values. (180° rotation).

> We also can say (0, 0, 0, 0) which is the "zero element", but it is not a valid rotation, because having w at 0 simply scaling the object down into singularity

The point on a sphere nature of the quaternions allows us to mix them by drawing a line between two points.

---

# Unity Cheat Sheet

## 1. Creating Rotations

How to convert human ideas into Quaternions.

|Goal|Code|Notes|
|---|---|---|
|**Reset Rotation**|`Quaternion.identity`|The "zero" rotation (0,0,0,1).|
|**Use Euler Angles**|`Quaternion.Euler(x, y, z)`|Most common. Converts (0-360) angles to Quaternion.|
|**Look at Target**|`Quaternion.LookRotation(direction)`|Points Z-axis (Blue) along the `direction` vector.|
|**Angle Axis**|`Quaternion.AngleAxis(angle, axis)`|Rotates `angle` degrees around `axis` (e.g., Vector3.up).|
|**Align Vectors**|`Quaternion.FromToRotation(from, to)`|Creates rotation needed to get from vector A to vector B.|

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
||||

## 4. Common Patterns

**Smooth Camera Look**

```csharp
// Slerp to a new target rotation
transform.rotation = Quaternion.Slerp(transform.rotation, targetRot, Time.deltaTime * speed);
```

**FPS Character Rotation (The Accumulator)**

```csharp
// Store angles in floats (not quaternions)
float yaw += Input.GetAxis("Mouse X");
float pitch -= Input.GetAxis("Mouse Y");
// Apply them all at once
transform.rotation = Quaternion.Euler(pitch, yaw, 0);
```

**Turret Tracking**

```csharp
Vector3 direction = target.position - transform.position;
Quaternion targetRot = Quaternion.LookRotation(direction);
transform.rotation = Quaternion.RotateTowards(transform.rotation, targetRot, turnSpeed * Time.deltaTime);
```

## The Forbidden Zone

- `myRot.x = 0.5f;` -> **NEVER** modify components directly.
- `myRot.eulerAngles.y += 10;` -> **AVOID** modifying eulerAngles property directly (drift issues).
- `rotA + rotB` -> **NONSENSE**. You multiply quaternions, you don't add them.

---

# Work Examples

Real scripts you'd actually use. Each one shows the **problem**, the **why** behind the quaternion choice, and the full working code.

---

## Example 1: FPS Camera Rig (Split Body/Head)

**Problem:** Player body rotates on Y (yaw) only. Camera on a child object pitches on X only. This is the standard setup for any first-person game.

**Why this structure:** Separating yaw and pitch onto different transforms means physics/character controller only deals with horizontal rotation. The camera never rolls, never gimbal-locks, and clamping pitch is trivial.

```csharp
public class FPSCameraRig : MonoBehaviour
{
    [SerializeField] Transform playerBody;   // The CharacterController object
    [SerializeField] Transform cameraHead;   // Child of playerBody, where Camera sits
    [SerializeField] float sensitivity = 2f;
    [SerializeField] float pitchClamp = 85f;

    float yaw;
    float pitch;

    void Start()
    {
        Cursor.lockState = CursorLockMode.Locked;
        // Initialize from current rotation so we don't snap on Start
        Vector3 currentEuler = playerBody.eulerAngles;
        yaw = currentEuler.y;
        pitch = 0f;
    }

    void Update()
    {
        float mouseX = Input.GetAxis("Mouse X") * sensitivity;
        float mouseY = Input.GetAxis("Mouse Y") * sensitivity;

        yaw += mouseX;
        pitch -= mouseY;
        pitch = Mathf.Clamp(pitch, -pitchClamp, pitchClamp);

        // Body only yaws — keeps CharacterController upright
        playerBody.rotation = Quaternion.Euler(0f, yaw, 0f);

        // Head only pitches — relative to body, so it inherits the yaw
        cameraHead.localRotation = Quaternion.Euler(pitch, 0f, 0f);
    }
}
```

> [!tip] Why `localRotation`? `cameraHead` is a child of `playerBody`. Setting `localRotation` means the pitch is _relative to the parent_. The parent already handles yaw, so the camera gets both without us multiplying anything manually.

---

## Example 2: Door That Opens On Interact

**Problem:** A door that smoothly swings open 90° when the player presses E, and swings back closed on another press.

**Why quaternions here:** We define two known rotations (open/closed) and Slerp between them. No angle tracking needed — just two target states.

```csharp
public class InteractDoor : MonoBehaviour
{
    [SerializeField] float openAngle = 90f;
    [SerializeField] float speed = 3f;

    Quaternion closedRot;
    Quaternion openRot;
    bool isOpen;

    void Start()
    {
        closedRot = transform.rotation;
        // Hinge rotates on Y axis
        openRot = closedRot * Quaternion.Euler(0f, openAngle, 0f);
    }

    void Update()
    {
        Quaternion target = isOpen ? openRot : closedRot;
        transform.rotation = Quaternion.Slerp(transform.rotation, target, Time.deltaTime * speed);
    }

    // Call this from your interaction system
    public void Toggle() => isOpen = !isOpen;
}
```

> [!info] Multiplication order `closedRot * Quaternion.Euler(0, 90, 0)` means "from the door's current orientation, rotate 90° around Y." If we did `Quaternion.Euler(0, 90, 0) * closedRot`, it would rotate 90° in **world** Y, which breaks if the door isn't axis-aligned.

---

## Example 3: Head Look / Aim IK (Simplified)

**Problem:** An NPC's head bone should track the player, but limited to a cone so it doesn't owl-neck 180°.

**Why `RotateTowards`:** It gives us a hard degree-per-frame cap, unlike Slerp which decelerates asymptotically and never truly arrives. The max angle check prevents unnatural neck snapping.

```csharp
public class HeadTrack : MonoBehaviour
{
    [SerializeField] Transform headBone;
    [SerializeField] Transform target;
    [SerializeField] float turnSpeed = 120f;  // degrees per second
    [SerializeField] float maxAngle = 70f;

    Quaternion restRotation;

    void Start()
    {
        restRotation = headBone.rotation;
    }

    // LateUpdate because we're modifying a bone AFTER the Animator has run
    void LateUpdate()
    {
        Vector3 dirToTarget = (target.position - headBone.position).normalized;
        Quaternion lookRot = Quaternion.LookRotation(dirToTarget, transform.up);

        // Clamp: if target is behind the NPC, return to rest
        float angle = Quaternion.Angle(restRotation, lookRot);
        Quaternion goalRot = angle <= maxAngle ? lookRot : restRotation;

        headBone.rotation = Quaternion.RotateTowards(
            headBone.rotation, goalRot, turnSpeed * Time.deltaTime
        );
    }
}
```

---

## Example 4: Recoil That Recovers

**Problem:** Gun kicks up on fire, then smoothly returns to where the player is looking. Common in FPS games.

**Why this works:** Recoil is stored as a separate offset from the player's actual aim. We layer it on top with multiplication, and decay it independently. The player's aim accumulators are never corrupted by the recoil.

```csharp
public class GunRecoil : MonoBehaviour
{
    [SerializeField] float recoilKickDeg = 5f;
    [SerializeField] float snapSpeed = 10f;   // how fast recoil applies
    [SerializeField] float returnSpeed = 6f;   // how fast it recovers

    Vector3 currentRecoil;   // where the recoil currently is
    Vector3 targetRecoil;    // where the recoil is going

    // Call this on Fire()
    public void AddRecoil()
    {
        float kickX = -recoilKickDeg;  // negative = up in most setups
        float kickY = Random.Range(-1f, 1f); // slight horizontal spread
        targetRecoil += new Vector3(kickX, kickY, 0f);
    }

    void Update()
    {
        // Snap toward the kick target
        currentRecoil = Vector3.Lerp(currentRecoil, targetRecoil, Time.deltaTime * snapSpeed);
        // Decay the target back to zero
        targetRecoil = Vector3.Lerp(targetRecoil, Vector3.zero, Time.deltaTime * returnSpeed);
    }

    /// <summary>
    /// Your camera script calls this and multiplies it on top of aim.
    /// cameraHead.localRotation = Quaternion.Euler(pitch, 0, 0) * recoil.GetRecoilRotation();
    /// </summary>
    public Quaternion GetRecoilRotation()
    {
        return Quaternion.Euler(currentRecoil);
    }
}
```

> [!warning] Don't bake recoil into pitch/yaw accumulators Some tutorials add recoil directly to the pitch float. This makes the camera "drift" permanently upward over sustained fire. Keeping recoil as a separate quaternion layer that decays to identity avoids this entirely.

---

## Example 5: Orbiting Camera (Third Person)

**Problem:** Camera orbits around the player at a fixed distance. Mouse controls the orbit angles. Camera should not clip through the ground.

**Why `Quaternion * Vector3`:** We use the quaternion to rotate an offset vector. Instead of doing trig (cos/sin) to position the camera on a sphere, we just rotate `Vector3.back * distance` by our desired orientation. The quaternion _is_ the sphere.

```csharp
public class OrbitCamera : MonoBehaviour
{
    [SerializeField] Transform focus;
    [SerializeField] float distance = 5f;
    [SerializeField] float sensitivity = 3f;
    [SerializeField] Vector2 pitchLimits = new(-20f, 70f);

    float yaw;
    float pitch = 20f;

    void LateUpdate()
    {
        yaw += Input.GetAxis("Mouse X") * sensitivity;
        pitch -= Input.GetAxis("Mouse Y") * sensitivity;
        pitch = Mathf.Clamp(pitch, pitchLimits.x, pitchLimits.y);

        Quaternion orbitRotation = Quaternion.Euler(pitch, yaw, 0f);

        // Key line: rotate "behind" vector by the orbit rotation
        Vector3 offset = orbitRotation * (Vector3.back * distance);

        transform.position = focus.position + offset;
        transform.rotation = orbitRotation;
    }
}
```

> [!tip] Why this is better than sin/cos `orbitRotation * Vector3.back` does the same thing as computing `(sin(yaw)*cos(pitch), sin(pitch), cos(yaw)*cos(pitch))` but it's readable, you don't get the signs wrong, and it works in any coordinate system.

---

## Example 6: Spaceship / 6DOF Flight

**Problem:** Full 3D rotation — pitch, yaw, _and_ roll. No "up" direction. Euler accumulators break here because of gimbal lock.

**Why pure quaternion accumulation:** There are no float accumulators. Each frame we build small rotation deltas and multiply them onto the existing rotation. This is the one case where you work _entirely_ in quaternions.

```csharp
public class SpaceshipController : MonoBehaviour
{
    [SerializeField] float pitchSpeed = 80f;
    [SerializeField] float yawSpeed = 60f;
    [SerializeField] float rollSpeed = 90f;
    [SerializeField] float thrust = 20f;

    void Update()
    {
        float dt = Time.deltaTime;
        float pitchInput = Input.GetAxis("Vertical");    // W/S
        float yawInput   = Input.GetAxis("Horizontal");  // A/D
        float rollInput  = Input.GetAxis("Roll");         // Q/E (custom axis)

        // Build incremental rotations around LOCAL axes
        Quaternion pitchDelta = Quaternion.AngleAxis(pitchInput * pitchSpeed * dt, Vector3.right);
        Quaternion yawDelta   = Quaternion.AngleAxis(yawInput   * yawSpeed   * dt, Vector3.up);
        Quaternion rollDelta  = Quaternion.AngleAxis(rollInput  * rollSpeed  * dt, Vector3.forward);

        // Multiply onto current rotation (right-multiply = local space)
        transform.rotation = transform.rotation * pitchDelta * yawDelta * rollDelta;

        // Thrust along local forward
        transform.position += transform.forward * thrust * dt;
    }
}
```

> [!info] Local vs World multiplication `transform.rotation * delta` applies `delta` in **local** space (ship turns relative to itself). `delta * transform.rotation` would apply it in **world** space (ship turns relative to the horizon — wrong for a spaceship, but useful for a compass needle).

---

## Example 7: Aligning To Surface Normal (Wall Running / Magnetism)

**Problem:** Player or object should align its "up" to match the surface they're on (wall running, walking on planets, magnetic boots).

**Why `FromToRotation`:** We need the rotation that takes "current up" to "surface normal." That's exactly what this function computes. Multiplying it onto the current rotation reorients the object without messing with its facing direction.

```csharp
public class SurfaceAlign : MonoBehaviour
{
    [SerializeField] float alignSpeed = 8f;
    [SerializeField] LayerMask groundMask;

    void Update()
    {
        if (Physics.Raycast(transform.position, -transform.up, out RaycastHit hit, 2f, groundMask))
        {
            // Rotation needed to go from our current up to the surface normal
            Quaternion alignDelta = Quaternion.FromToRotation(transform.up, hit.normal);

            // Apply it to our current rotation
            Quaternion targetRot = alignDelta * transform.rotation;

            transform.rotation = Quaternion.Slerp(transform.rotation, targetRot, Time.deltaTime * alignSpeed);
        }
    }
}
```

---

## Example 8: Inverse — "How Far Did I Turn?"

**Problem:** You need to know the _relative_ rotation between two orientations. For example: detecting if a VR player physically turned their head more than 30° from a reference direction, or calculating how much a steering wheel has rotated from center.

**Why `Inverse`:** `Inverse(A) * B` gives you "the rotation that gets from A to B." It's subtraction for quaternions.

```csharp
public class TurnDetector : MonoBehaviour
{
    Quaternion referenceRotation;

    public void SetReference()
    {
        referenceRotation = transform.rotation;
    }

    public float GetDegreesFromReference()
    {
        // "Undo" the reference, then see what's left
        Quaternion delta = Quaternion.Inverse(referenceRotation) * transform.rotation;
        float angle = Quaternion.Angle(Quaternion.identity, delta);
        return angle;
    }

    // Usage: check if player has looked away
    void Update()
    {
        if (GetDegreesFromReference() > 30f)
        {
            Debug.Log("Player looked away more than 30°");
        }
    }
}
```

---

## Decision Flowchart: Which Tool Do I Use?

```
Is rotation constrained to 1-2 axes (FPS, top-down, 2D)?
├─ YES → Use float accumulators + Quaternion.Euler() to build final rotation
│         (Examples 1, 5)
│
└─ NO → Is it full 3D freedom (spaceship, 6DOF)?
   ├─ YES → Accumulate with quaternion multiplication, no floats
   │         (Example 6)
   │
   └─ NO → Do you have a known start and end state?
      ├─ YES → Slerp or RotateTowards between them
      │         (Examples 2, 3)
      │
      └─ NO → Do you need to align one direction to another?
         ├─ YES → FromToRotation or LookRotation
         │         (Examples 3, 7)
         │
         └─ Do you need the difference between two rotations?
            └─ YES → Inverse(A) * B
                      (Example 8)
```