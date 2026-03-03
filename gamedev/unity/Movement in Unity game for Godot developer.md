# Movement in Unity

Movement APIs in Unity: CharacterController, Rigidbody, and Transform-based movement with Godot equivalents.

---

## Movement Approaches Overview

| Unity Approach | Godot Equivalent | Physics? | Gravity auto? |
|---|---|---|---|
| `CharacterController` | `CharacterBody3D.move_and_slide()` | Built-in collision, no full physics | No |
| `Rigidbody` + forces | `RigidBody3D.apply_force()` | Full physics simulation | Yes |
| `Rigidbody.MovePosition()` | `RigidBody3D` with `AnimatableBody3D` | Kinematic-style physics | Yes |
| `Transform.Translate()` | `Node3D.translate()` / `position +=` | None | No |

---

## CharacterController

Requires `CharacterController` component on the GameObject. Does not use Rigidbody.

### Properties

| Property | Type | Description |
|---|---|---|
| `isGrounded` | `bool` | True if last `Move()` touched ground |
| `velocity` | `Vector3` | Current velocity from last `Move()` call |
| `height` | `float` | Capsule height |
| `radius` | `float` | Capsule radius |
| `center` | `Vector3` | Capsule center offset |
| `slopeLimit` | `float` | Max walkable slope in degrees |
| `stepOffset` | `float` | Max step height in meters |
| `minMoveDistance` | `float` | Minimum movement distance (avoids jitter) |
| `skinWidth` | `float` | Overlap tolerance for collisions |

### Move()

```csharp
// CollisionFlags Move(Vector3 motion)
// Moves the controller by motion. No gravity applied automatically.
// Call in Update(). motion is NOT framerate-independent by itself.

Vector3 velocity = Vector3.zero;
velocity.x = input.x * speed;
velocity.z = input.y * speed;
velocity.y += gravity * Time.deltaTime; // manual gravity

CollisionFlags flags = controller.Move(velocity * Time.deltaTime);
```

**Return value — `CollisionFlags`:**

| Flag | Value |
|---|---|
| `CollisionFlags.None` | 0 |
| `CollisionFlags.Sides` | 1 |
| `CollisionFlags.Above` | 2 |
| `CollisionFlags.Below` | 4 |

Godot equivalent: `CharacterBody3D.move_and_slide()` (gravity also manual, uses `velocity` property directly).

### SimpleMove()

```csharp
// bool SimpleMove(Vector3 speed)
// Applies gravity automatically. Ignores Y component of speed.
// Call in Update(). Framerate-independent internally (applies Time.deltaTime).

controller.SimpleMove(direction * speed);
```

| | `Move()` | `SimpleMove()` |
|---|---|---|
| Gravity | Manual | Automatic |
| Y-axis control | Full | Ignored |
| Time.deltaTime | Must apply yourself | Applied internally |
| Return | `CollisionFlags` | `bool` (was grounded) |
| Jumping | Possible | Not possible (Y ignored) |

---

## Rigidbody Movement

Requires `Rigidbody` component. Physics handled by PhysicsEngine.

### AddForce()

```csharp
// void AddForce(Vector3 force, ForceMode mode = ForceMode.Force)
// Call in FixedUpdate().

rb.AddForce(Vector3.forward * 10f, ForceMode.Force);
rb.AddForce(Vector3.up * jumpForce, ForceMode.Impulse);
```

**ForceMode enum:**

| ForceMode | Effect | Mass-dependent? | Godot equivalent |
|---|---|---|---|
| `Force` | Continuous force (N), per-frame accumulation | Yes | `apply_force()` |
| `Acceleration` | Continuous acceleration (m/s^2) | No | `apply_force() / mass` |
| `Impulse` | Instant velocity change (N*s) | Yes | `apply_impulse()` |
| `VelocityChange` | Instant velocity change (m/s) | No | `linear_velocity +=` |

### AddRelativeForce()

```csharp
// void AddRelativeForce(Vector3 force, ForceMode mode = ForceMode.Force)
// Force relative to the Rigidbody's local coordinate system.

rb.AddRelativeForce(Vector3.forward * 10f);
```

### AddTorque()

```csharp
// void AddTorque(Vector3 torque, ForceMode mode = ForceMode.Force)
rb.AddTorque(Vector3.up * rotationForce);
```

### MovePosition()

```csharp
// void MovePosition(Vector3 position)
// Interpolates to target position. Use with isKinematic = true for kinematic,
// or with dynamic Rigidbody for physics-respecting movement.
// Call in FixedUpdate().

rb.MovePosition(rb.position + direction * speed * Time.fixedDeltaTime);
```

Godot equivalent: Setting `CharacterBody3D.velocity` then calling `move_and_slide()`, or `AnimatableBody3D`.

### MoveRotation()

```csharp
// void MoveRotation(Quaternion rot)
// Call in FixedUpdate().

Quaternion target = Quaternion.Euler(0, angle, 0);
rb.MoveRotation(target);
```

### Direct Velocity

```csharp
// Directly set velocity (overrides physics forces)
rb.linearVelocity = new Vector3(moveX, rb.linearVelocity.y, moveZ);
```

> [!warning]
> `rb.velocity` is deprecated in Unity 6+. Use `rb.linearVelocity` and `rb.angularVelocity` instead.

Godot equivalent: `rigid_body.linear_velocity = Vector3(...)`.

### Rigidbody Properties

| Property | Type | Description |
|---|---|---|
| `mass` | `float` | Mass in kg |
| `linearDamping` | `float` | Air resistance (was `drag`) |
| `angularDamping` | `float` | Rotational resistance (was `angularDrag`) |
| `useGravity` | `bool` | Affected by gravity |
| `isKinematic` | `bool` | Not driven by physics engine |
| `interpolation` | `RigidbodyInterpolation` | None, Interpolate, Extrapolate |
| `collisionDetectionMode` | `CollisionDetectionMode` | Discrete, Continuous, ContinuousDynamic, ContinuousSpeculative |
| `constraints` | `RigidbodyConstraints` | Freeze position/rotation axes |
| `linearVelocity` | `Vector3` | Current linear velocity |
| `angularVelocity` | `Vector3` | Current angular velocity |

---

## Transform Movement (No Physics)

No collision detection. No physics interaction.

### Translate()

```csharp
// void Translate(Vector3 translation, Space relativeTo = Space.Self)
// void Translate(Vector3 translation, Transform relativeTo)

transform.Translate(Vector3.forward * speed * Time.deltaTime);              // local space
transform.Translate(Vector3.forward * speed * Time.deltaTime, Space.World); // world space
transform.Translate(Vector3.forward * speed * Time.deltaTime, camera.transform); // relative to camera
```

Godot equivalent: `translate(Vector3)` for local, `global_position += Vector3` for world.

### Direct Position

```csharp
// Teleport / instant move
transform.position = new Vector3(0, 5, 0);           // world
transform.localPosition = new Vector3(0, 5, 0);      // relative to parent
transform.position += direction * speed * Time.deltaTime; // incremental
```

Godot equivalent: `global_position = Vector3(...)` or `position = Vector3(...)`.

### Rotation

```csharp
// void Rotate(Vector3 eulers, Space relativeTo = Space.Self)
transform.Rotate(0, rotationSpeed * Time.deltaTime, 0);
transform.Rotate(Vector3.up, rotationSpeed * Time.deltaTime, Space.World);

// Look at target
transform.LookAt(target.position);
transform.LookAt(target.position, Vector3.up);  // specify up direction
```

Godot equivalent: `rotate_y(angle)`, `look_at(target, up)`.

---

## Timing Rules

| Method | Call in | Use this delta |
|---|---|---|
| `CharacterController.Move()` | `Update()` | `Time.deltaTime` |
| `CharacterController.SimpleMove()` | `Update()` | None (internal) |
| `Rigidbody.AddForce()` | `FixedUpdate()` | None (physics handles it) |
| `Rigidbody.MovePosition()` | `FixedUpdate()` | `Time.fixedDeltaTime` |
| `Transform.Translate()` | `Update()` | `Time.deltaTime` |
| `Transform.position +=` | `Update()` | `Time.deltaTime` |

---

## NavMeshAgent (AI Movement)

```csharp
// UnityEngine.AI.NavMeshAgent
agent.SetDestination(targetPosition);  // pathfind to position
agent.speed = 3.5f;
agent.stoppingDistance = 1f;
agent.isStopped = true;                // pause movement
agent.ResetPath();                     // clear current path
agent.remainingDistance;                // float
agent.hasPath;                         // bool
agent.pathStatus;                      // NavMeshPathStatus
```

Godot equivalent: `NavigationAgent3D` with `target_position` property.

---

## Sources
- [Unity Scripting API: CharacterController](https://docs.unity3d.com/ScriptReference/CharacterController.html)
- [Unity Scripting API: Rigidbody](https://docs.unity3d.com/ScriptReference/Rigidbody.html)
- [Unity Scripting API: Rigidbody2D](https://docs.unity3d.com/ScriptReference/Rigidbody2D.html)
- [Unity Scripting API: Transform](https://docs.unity3d.com/ScriptReference/Transform.html)
- [Unity Scripting API: Physics](https://docs.unity3d.com/ScriptReference/Physics.html)
- [Unity Manual: NavMeshAgent](https://docs.unity3d.com/ScriptReference/AI.NavMeshAgent.html)

## Related topics
- [[MonoBehaviour Reference]]
- [[Execution Order in Unity]]
- [[GameObject and Component Model]]
