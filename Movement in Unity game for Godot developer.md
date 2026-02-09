# Player Movement in Unity

> [!quote] The Core Insight Unity gives you five ways to move things. The internet will show you all five with no guidance. Your game type determines the answer.

---

## 1. Why Movement in Unity Is Confusing

In Godot, you pick a node type and movement is built in:

- `CharacterBody3D` → `move_and_slide()` handles everything
- `CharacterBody2D` → same in 2D
- `RigidBody3D` → physics handles it

Unity doesn't work this way. There's no `CharacterBody` equivalent that "just works." Instead, you combine a physics component (or not) with your own movement code. This gives you more control but forces you to make decisions Godot made for you.

Here are the movement approaches you'll encounter:

|Approach|How It Works|Physics?|Godot Equivalent|
|---|---|---|---|
|`transform.Translate()`|Teleport every frame|None|`position += velocity * delta`|
|`CharacterController.Move()`|Sweep + slide, no physics|Collision only|Closest to `move_and_slide()`|
|`Rigidbody.velocity = ...`|Direct velocity control|Partial (gravity, collisions)|`CharacterBody3D` with manual velocity|
|`Rigidbody.AddForce()`|Apply force, physics resolves|Full|`RigidBody3D.apply_force()`|
|`Rigidbody.MovePosition()`|Kinematic move with collision|Collision only|`AnimatableBody3D`|

**Every tutorial you watch will use a different one.** That's not because the others are wrong — it's because they're solving different problems. The rest of this document tells you which to pick.

---

## 2. The Decision Tree

```
What kind of game are you making?

2D Platformer
├── Want tight, precise control (Celeste, Hollow Knight)?
│   → Rigidbody2D + direct velocity (Section 4)
└── Want physics-heavy feel (Angry Birds, physics puzzler)?
    → Rigidbody2D + AddForce (Section 4)

2D Top-Down (Zelda-like, twin-stick)
└── → Rigidbody2D + velocity or MovePosition (Section 5)

3D First Person
├── Classic FPS (Quake, Half-Life)?
│   → CharacterController (Section 6)
└── Physics interactions needed (pushing, knockback, riding platforms)?
    → Rigidbody + velocity (Section 6)

3D Third Person
├── Action/adventure (Souls-like, Zelda)?
│   → CharacterController or Rigidbody (Section 7)
└── Needs root motion (animation-driven movement)?
    → Rigidbody + Animator root motion (Section 7)

AI / NPCs
└── → NavMeshAgent (not covered here — separate topic)
```

---

## 3. Universal Rules (All Approaches)

Before diving into specific setups, these rules apply to every movement approach:

### Input in Update, Movement in FixedUpdate

```csharp
// Input MUST be polled in Update (runs every frame)
// Physics MUST happen in FixedUpdate (runs at fixed rate)

private Vector2 _inputDirection;

private void Update()
{
    // Capture input every frame — FixedUpdate can miss button presses
    _inputDirection = new Vector2(Input.GetAxisRaw("Horizontal"), Input.GetAxisRaw("Vertical"));
    
    // For one-shot actions, set a flag
    if (Input.GetButtonDown("Jump"))
        _jumpRequested = true;
}

private void FixedUpdate()
{
    // Use the captured input for physics movement
    Move(_inputDirection);
    
    if (_jumpRequested)
    {
        Jump();
        _jumpRequested = false;
    }
}
```

**Exception:** `CharacterController.Move()` runs in `Update` (it's not physics-based).

### Ground Detection

Every approach needs to know if the player is on the ground (for jumping, animations, etc.):

```csharp
// 3D — SphereCast downward from feet
[SerializeField] private Transform _groundCheck;  // Empty child at character's feet
[SerializeField] private float _groundDistance = 0.2f;
[SerializeField] private LayerMask _groundMask;

private bool _isGrounded;

private void CheckGround()
{
    _isGrounded = Physics.CheckSphere(_groundCheck.position, _groundDistance, _groundMask);
}
```

```csharp
// 2D — OverlapCircle downward
[SerializeField] private Transform _groundCheck;
[SerializeField] private float _groundDistance = 0.1f;
[SerializeField] private LayerMask _groundMask;

private bool _isGrounded;

private void CheckGround()
{
    _isGrounded = Physics2D.OverlapCircle(_groundCheck.position, _groundDistance, _groundMask);
}
```

Place `_groundCheck` as a child GameObject slightly below the collider's bottom.

### Delta Time — Which One?

|Context|Use|Value|
|---|---|---|
|`Update()`|`Time.deltaTime`|Varies per frame|
|`FixedUpdate()`|`Time.fixedDeltaTime`|Constant (default 0.02)|
|`Rigidbody` forces in `FixedUpdate`|Already frame-rate independent|Don't multiply by delta|

> [!warning] `AddForce()` with `ForceMode.Force` is already multiplied by time internally. Don't double-multiply. `ForceMode.VelocityChange` and `ForceMode.Impulse` are instant — also don't multiply.

### Visual Smoothing

Physics runs at a fixed rate (50Hz default). The camera renders at the display rate (60Hz+). Without interpolation, movement looks stuttery.

```csharp
// On the Rigidbody component in Inspector:
// Interpolation → Interpolate (smooths between physics steps)
// This is the #1 fix for jittery movement. Always enable for the player.
```

For `CharacterController` (no Rigidbody), movement happens in `Update` so this isn't an issue.

---

## 4. 2D Platformer Movement

### Setup

```
GameObject "Player"
├── SpriteRenderer
├── CapsuleCollider2D (or BoxCollider2D)
├── Rigidbody2D (Dynamic, Freeze Rotation Z)
├── Animator
└── PlayerMovement2D.cs
```

**Rigidbody2D settings:**

- Body Type: `Dynamic`
- Gravity Scale: `2-4` (default 1 feels floaty — tune this!)
- Freeze Rotation Z: ✅ (prevents tumbling)
- Collision Detection: `Continuous` (if moving fast)
- Interpolation: `Interpolate`

### The Three Approaches (and When to Use Each)

#### Approach A: Direct Velocity (Recommended for most platformers)

The most common approach for tight, responsive platformers. You directly set horizontal velocity and let physics handle gravity.

```csharp
[RequireComponent(typeof(Rigidbody2D))]
public class PlatformerMovement : MonoBehaviour
{
    [Header("Movement")]
    [SerializeField] private float _moveSpeed = 8f;
    [SerializeField] private float _jumpForce = 12f;
    
    [Header("Ground Check")]
    [SerializeField] private Transform _groundCheck;
    [SerializeField] private float _groundRadius = 0.1f;
    [SerializeField] private LayerMask _groundMask;
    
    private Rigidbody2D _rb;
    private float _moveInput;
    private bool _jumpRequested;
    private bool _isGrounded;
    
    private void Awake()
    {
        _rb = GetComponent<Rigidbody2D>();
    }
    
    private void Update()
    {
        _moveInput = Input.GetAxisRaw("Horizontal");
        
        if (Input.GetButtonDown("Jump") && _isGrounded)
            _jumpRequested = true;
    }
    
    private void FixedUpdate()
    {
        _isGrounded = Physics2D.OverlapCircle(
            _groundCheck.position, _groundRadius, _groundMask);
        
        // Horizontal: direct velocity control (responsive, no sliding)
        _rb.velocity = new Vector2(_moveInput * _moveSpeed, _rb.velocity.y);
        
        // Vertical: impulse for jump, gravity handles the rest
        if (_jumpRequested)
        {
            _rb.velocity = new Vector2(_rb.velocity.x, _jumpForce);
            _jumpRequested = false;
        }
    }
}
```

**Why this works:** Horizontal velocity is set directly — movement starts and stops instantly, no acceleration ramp. Vertical velocity is only touched for jumping — gravity handles falling naturally. This is what Celeste, Hollow Knight, and most precision platformers do under the hood.

**The tradeoff:** Setting `velocity.x` directly overwrites external horizontal forces (knockback, wind, conveyor belts). See Approach C below.

#### Approach B: AddForce (Physics-heavy feel)

Movement feels heavy, momentum-based. Character slides, accelerates gradually.

```csharp
private void FixedUpdate()
{
    // Force-based: character accelerates, has momentum
    _rb.AddForce(new Vector2(_moveInput * _acceleration, 0f));
    
    // Clamp max speed manually
    if (Mathf.Abs(_rb.velocity.x) > _maxSpeed)
        _rb.velocity = new Vector2(Mathf.Sign(_rb.velocity.x) * _maxSpeed, _rb.velocity.y);
    
    // Jump
    if (_jumpRequested)
    {
        _rb.AddForce(Vector2.up * _jumpForce, ForceMode2D.Impulse);
        _jumpRequested = false;
    }
}
```

**When to use:** Physics puzzlers, games where weight and momentum are part of the feel. Not ideal for precision platformers — characters feel "slippery."

#### Approach C: Hybrid (Best of both — recommended for complex games)

Direct control for player-initiated movement, but respects external forces:

```csharp
private void FixedUpdate()
{
    _isGrounded = Physics2D.OverlapCircle(
        _groundCheck.position, _groundRadius, _groundMask);
    
    // Calculate desired velocity
    float targetVelocityX = _moveInput * _moveSpeed;
    
    // Smoothly approach target (allows external forces to influence)
    float velocityChangeX = targetVelocityX - _rb.velocity.x;
    
    // Higher value = snappier response, lower = slidier
    float smoothing = _isGrounded ? 0.8f : 0.4f;  // Less control in air
    
    _rb.velocity += new Vector2(velocityChangeX * smoothing, 0f);
    
    if (_jumpRequested)
    {
        _rb.velocity = new Vector2(_rb.velocity.x, _jumpForce);
        _jumpRequested = false;
    }
}
```

**Why this is good:** The character still has responsive control but external forces (knockback, explosions) can push them because you're adjusting velocity rather than overwriting it.

### Common Platformer Enhancements

**Coyote Time** — allow jumping for a few frames after leaving a ledge:

```csharp
private float _coyoteTimer;
private const float CoyoteTime = 0.1f;

private void FixedUpdate()
{
    if (_isGrounded)
        _coyoteTimer = CoyoteTime;
    else
        _coyoteTimer -= Time.fixedDeltaTime;
    
    bool canJump = _coyoteTimer > 0f;
    
    if (_jumpRequested && canJump)
    {
        _rb.velocity = new Vector2(_rb.velocity.x, _jumpForce);
        _coyoteTimer = 0f;  // Consume coyote time
        _jumpRequested = false;
    }
}
```

**Jump Buffering** — remember jump press slightly before landing:

```csharp
private float _jumpBufferTimer;
private const float JumpBufferTime = 0.15f;

private void Update()
{
    if (Input.GetButtonDown("Jump"))
        _jumpBufferTimer = JumpBufferTime;
    else
        _jumpBufferTimer -= Time.deltaTime;
}

private void FixedUpdate()
{
    if (_jumpBufferTimer > 0f && _isGrounded)
    {
        _rb.velocity = new Vector2(_rb.velocity.x, _jumpForce);
        _jumpBufferTimer = 0f;
    }
}
```

**Variable Jump Height** — release jump early for short hop:

```csharp
private void Update()
{
    // Cut jump short when button released
    if (Input.GetButtonUp("Jump") && _rb.velocity.y > 0f)
    {
        _rb.velocity = new Vector2(_rb.velocity.x, _rb.velocity.y * 0.5f);
    }
}
```

**Better Gravity** — fall faster than rise (game feel staple):

```csharp
[SerializeField] private float _fallMultiplier = 2.5f;
[SerializeField] private float _lowJumpMultiplier = 2f;

private void FixedUpdate()
{
    // ... movement code ...
    
    // Better jump arc
    if (_rb.velocity.y < 0)  // Falling
    {
        _rb.velocity += Vector2.up * Physics2D.gravity.y * (_fallMultiplier - 1) * Time.fixedDeltaTime;
    }
    else if (_rb.velocity.y > 0 && !Input.GetButton("Jump"))  // Rising but released
    {
        _rb.velocity += Vector2.up * Physics2D.gravity.y * (_lowJumpMultiplier - 1) * Time.fixedDeltaTime;
    }
}
```

### Godot Comparison

|Godot|Unity (Direct Velocity)|
|---|---|
|`velocity.x = direction * speed`|`rb.velocity = new Vector2(input * speed, rb.velocity.y)`|
|`velocity.y = jump_force`|`rb.velocity = new Vector2(rb.velocity.x, jumpForce)`|
|`velocity = move_and_slide(velocity)`|Physics engine handles this automatically|
|`is_on_floor()`|`Physics2D.OverlapCircle(...)` (manual check)|
|Gravity is in `_physics_process`|Rigidbody2D handles gravity automatically|

The biggest difference: Godot's `move_and_slide()` does collision resolution for you. In Unity, the Rigidbody2D + Collider does it, but you set velocity and the physics engine resolves collisions.

---

## 5. 2D Top-Down Movement

### Setup

```
GameObject "Player"
├── SpriteRenderer
├── CircleCollider2D (or CapsuleCollider2D)
├── Rigidbody2D (Dynamic, Gravity Scale = 0, Freeze Rotation Z)
├── Animator
└── TopDownMovement.cs
```

**Rigidbody2D settings:**

- Body Type: `Dynamic`
- Gravity Scale: `0` (top-down has no gravity)
- Freeze Rotation Z: ✅
- Linear Drag: `5-10` (prevents infinite sliding — tune to feel)
- Interpolation: `Interpolate`

### Direct Velocity (Responsive)

```csharp
[RequireComponent(typeof(Rigidbody2D))]
public class TopDownMovement : MonoBehaviour
{
    [SerializeField] private float _moveSpeed = 5f;
    
    private Rigidbody2D _rb;
    private Vector2 _moveInput;
    
    private void Awake()
    {
        _rb = GetComponent<Rigidbody2D>();
    }
    
    private void Update()
    {
        _moveInput = new Vector2(
            Input.GetAxisRaw("Horizontal"),
            Input.GetAxisRaw("Vertical")
        ).normalized;  // Prevent diagonal speed boost
    }
    
    private void FixedUpdate()
    {
        _rb.velocity = _moveInput * _moveSpeed;
    }
}
```

**Important:** `.normalized` prevents diagonal movement from being ~41% faster than cardinal movement (Pythagorean theorem: √(1²+1²) = 1.414).

### MovePosition (Alternative — Kinematic style)

```csharp
private void FixedUpdate()
{
    Vector2 targetPos = _rb.position + _moveInput * _moveSpeed * Time.fixedDeltaTime;
    _rb.MovePosition(targetPos);
}
```

Same feel, but uses `MovePosition` which respects colliders without using velocity. Good when you don't want any sliding or momentum.

---

## 6. 3D First-Person Movement

### Option A: CharacterController (Recommended starting point)

The `CharacterController` is Unity's closest equivalent to Godot's `CharacterBody3D`. No real physics — just sweep, collide, and slide. You handle gravity yourself.

#### Setup

```
GameObject "Player"
├── CharacterController (radius 0.5, height 2, center 0,1,0)
├── Main Camera (child, positioned at head height)
└── FPSController.cs
```

No Rigidbody needed. The CharacterController IS the collider.

#### Code

```csharp
[RequireComponent(typeof(CharacterController))]
public class FPSController : MonoBehaviour
{
    [Header("Movement")]
    [SerializeField] private float _walkSpeed = 6f;
    [SerializeField] private float _sprintSpeed = 10f;
    [SerializeField] private float _jumpHeight = 1.2f;
    [SerializeField] private float _gravity = -20f;
    
    [Header("Mouse Look")]
    [SerializeField] private float _mouseSensitivity = 2f;
    [SerializeField] private Transform _cameraTransform;
    
    private CharacterController _cc;
    private Vector3 _velocity;
    private float _cameraPitch;
    
    private void Awake()
    {
        _cc = GetComponent<CharacterController>();
        Cursor.lockState = CursorLockMode.Locked;
    }
    
    private void Update()
    {
        HandleMouseLook();
        HandleMovement();
    }
    
    private void HandleMouseLook()
    {
        float mouseX = Input.GetAxis("Mouse X") * _mouseSensitivity;
        float mouseY = Input.GetAxis("Mouse Y") * _mouseSensitivity;
        
        // Horizontal rotation — rotate the whole player
        transform.Rotate(Vector3.up, mouseX);
        
        // Vertical rotation — rotate only the camera
        _cameraPitch -= mouseY;
        _cameraPitch = Mathf.Clamp(_cameraPitch, -90f, 90f);
        _cameraTransform.localRotation = Quaternion.Euler(_cameraPitch, 0f, 0f);
    }
    
    private void HandleMovement()
    {
        // Ground check (built into CharacterController)
        if (_cc.isGrounded && _velocity.y < 0f)
            _velocity.y = -2f;  // Small downward force to stay grounded
        
        // Input
        float x = Input.GetAxisRaw("Horizontal");
        float z = Input.GetAxisRaw("Vertical");
        
        // Move relative to where we're facing
        Vector3 move = transform.right * x + transform.forward * z;
        move = Vector3.ClampMagnitude(move, 1f);  // Prevent diagonal speed boost
        
        float speed = Input.GetKey(KeyCode.LeftShift) ? _sprintSpeed : _walkSpeed;
        _cc.Move(move * speed * Time.deltaTime);
        
        // Jump
        if (Input.GetButtonDown("Jump") && _cc.isGrounded)
        {
            // v = sqrt(2 * |gravity| * jumpHeight)
            _velocity.y = Mathf.Sqrt(_jumpHeight * -2f * _gravity);
        }
        
        // Gravity (manual — CharacterController has no built-in gravity)
        _velocity.y += _gravity * Time.deltaTime;
        _cc.Move(_velocity * Time.deltaTime);
    }
}
```

**Key differences from Godot:**

- `move_and_slide()` handles gravity internally. `CharacterController.Move()` doesn't — you apply gravity yourself.
- `is_on_floor()` → `_cc.isGrounded` (same concept, built-in)
- `velocity` isn't a property on CharacterController — you track it yourself
- Mouse look is fully manual (Godot's `Input.get_last_mouse_velocity()` vs Unity's `Input.GetAxis("Mouse X")`)

#### CharacterController Properties

|Property|What It Does|Typical Value|
|---|---|---|
|`Height`|Capsule height|2 (standing human)|
|`Radius`|Capsule radius|0.5|
|`Center`|Capsule center offset|(0, 1, 0) for feet at origin|
|`Slope Limit`|Max walkable angle (degrees)|45|
|`Step Offset`|Max step height to climb|0.3|
|`Skin Width`|Collision padding|0.08 (keep > 0)|
|`Min Move Distance`|Ignore moves smaller than this|0.001|

### Option B: Rigidbody-Based FPS (Physics interactions needed)

Use this when the player needs to be pushed by explosions, ride physics platforms, or interact physically with the world.

#### Setup

```
GameObject "Player"
├── CapsuleCollider (radius 0.5, height 2, center 0,1,0)
├── Rigidbody (Freeze Rotation X/Y/Z, Interpolate, Continuous collision)
├── Main Camera (child, at head height)
└── FPSRigidbodyController.cs
```

#### Code

```csharp
[RequireComponent(typeof(Rigidbody))]
public class FPSRigidbodyController : MonoBehaviour
{
    [Header("Movement")]
    [SerializeField] private float _moveSpeed = 6f;
    [SerializeField] private float _jumpForce = 7f;
    
    [Header("Ground Check")]
    [SerializeField] private Transform _groundCheck;
    [SerializeField] private float _groundDistance = 0.3f;
    [SerializeField] private LayerMask _groundMask;
    
    [Header("Mouse Look")]
    [SerializeField] private float _mouseSensitivity = 2f;
    [SerializeField] private Transform _cameraTransform;
    
    private Rigidbody _rb;
    private Vector3 _moveInput;
    private bool _jumpRequested;
    private bool _isGrounded;
    private float _cameraPitch;
    
    private void Awake()
    {
        _rb = GetComponent<Rigidbody>();
        _rb.freezeRotation = true;
        Cursor.lockState = CursorLockMode.Locked;
    }
    
    private void Update()
    {
        // Input (every frame)
        float x = Input.GetAxisRaw("Horizontal");
        float z = Input.GetAxisRaw("Vertical");
        _moveInput = (transform.right * x + transform.forward * z);
        _moveInput = Vector3.ClampMagnitude(_moveInput, 1f);
        
        if (Input.GetButtonDown("Jump") && _isGrounded)
            _jumpRequested = true;
        
        // Mouse look (every frame for smoothness)
        float mouseX = Input.GetAxis("Mouse X") * _mouseSensitivity;
        float mouseY = Input.GetAxis("Mouse Y") * _mouseSensitivity;
        transform.Rotate(Vector3.up, mouseX);
        _cameraPitch -= mouseY;
        _cameraPitch = Mathf.Clamp(_cameraPitch, -90f, 90f);
        _cameraTransform.localRotation = Quaternion.Euler(_cameraPitch, 0f, 0f);
    }
    
    private void FixedUpdate()
    {
        _isGrounded = Physics.CheckSphere(_groundCheck.position, _groundDistance, _groundMask);
        
        // Movement: set horizontal velocity directly, preserve vertical (gravity)
        Vector3 targetVelocity = _moveInput * _moveSpeed;
        targetVelocity.y = _rb.velocity.y;  // Don't override gravity
        _rb.velocity = targetVelocity;
        
        // Jump
        if (_jumpRequested)
        {
            _rb.velocity = new Vector3(_rb.velocity.x, _jumpForce, _rb.velocity.z);
            _jumpRequested = false;
        }
    }
}
```

**Why freeze rotation?** Without it, the capsule collider tips over when hitting walls. Freezing rotation on all axes keeps the player upright, and you handle rotation yourself via mouse look.

### CharacterController vs Rigidbody — Decision

|Factor|CharacterController|Rigidbody|
|---|---|---|
|**Ease of use**|Simpler — no physics to fight|More setup, physics quirks|
|**Gravity**|Manual (you write it)|Automatic|
|**Slopes/Steps**|Built-in (`Slope Limit`, `Step Offset`)|Manual (or workarounds)|
|**Pushed by physics objects**|No (unless you code it)|Yes, naturally|
|**Knockback, explosions**|Must fake with your own velocity|Works via AddForce|
|**Moving platforms**|Works (parenting or manual)|Can be tricky (friction, sliding)|
|**isGrounded**|Built-in|Manual (raycast/overlap)|
|**OnCollisionEnter events**|No (uses OnControllerColliderHit)|Yes|
|**Runs in**|Update|FixedUpdate|

**Rule of thumb:** Start with `CharacterController`. Switch to Rigidbody when you hit a wall trying to add physics interactions.

---

## 7. 3D Third-Person Movement

Third-person adds complexity: the character faces a direction separate from the camera, and rotation feels different.

### Setup

```
GameObject "Player"
├── Character Model (child, with Animator)
├── CapsuleCollider (or CharacterController)
├── Rigidbody (if physics-based)
└── ThirdPersonController.cs

GameObject "CameraRig" (separate from player!)
├── Cinemachine FreeLook Camera (or your own follow camera)
└── targets the Player
```

> [!tip] **Camera:** Don't parent the camera to the player in third-person. Use Cinemachine FreeLook or your own smooth-follow script. Parented cameras jitter and can't orbit.

### Movement Relative to Camera

The key difference from first-person: input is relative to the **camera**, not the player. When you press "forward," the player moves away from the camera, regardless of which way the player model is facing.

```csharp
public class ThirdPersonMovement : MonoBehaviour
{
    [SerializeField] private float _moveSpeed = 6f;
    [SerializeField] private float _rotationSpeed = 10f;
    [SerializeField] private Transform _cameraTransform;
    
    private CharacterController _cc;
    private Vector3 _velocity;
    private float _gravity = -20f;
    
    private void Awake()
    {
        _cc = GetComponent<CharacterController>();
    }
    
    private void Update()
    {
        if (_cc.isGrounded && _velocity.y < 0f)
            _velocity.y = -2f;
        
        float x = Input.GetAxisRaw("Horizontal");
        float z = Input.GetAxisRaw("Vertical");
        
        // Direction relative to CAMERA (not player)
        Vector3 camForward = _cameraTransform.forward;
        Vector3 camRight = _cameraTransform.right;
        camForward.y = 0f;  // Flatten to horizontal plane
        camRight.y = 0f;
        camForward.Normalize();
        camRight.Normalize();
        
        Vector3 moveDir = (camForward * z + camRight * x);
        moveDir = Vector3.ClampMagnitude(moveDir, 1f);
        
        if (moveDir.magnitude > 0.1f)
        {
            // Rotate character to face movement direction
            Quaternion targetRotation = Quaternion.LookRotation(moveDir);
            transform.rotation = Quaternion.Slerp(
                transform.rotation, targetRotation, _rotationSpeed * Time.deltaTime);
            
            // Move
            _cc.Move(moveDir * _moveSpeed * Time.deltaTime);
        }
        
        // Gravity
        _velocity.y += _gravity * Time.deltaTime;
        _cc.Move(_velocity * Time.deltaTime);
    }
}
```

**The camera-relative trick:** Project camera's forward/right onto the horizontal plane, normalize, then use as the movement basis. This is why pressing "W" always moves the character away from the camera.

### Root Motion (Animation-Driven Movement)

For games where movement speed should match animation exactly (souls-likes, realistic characters):

```csharp
// On the Animator component:
// Apply Root Motion: ✅

// In code — the Animator drives movement, you just provide input
[RequireComponent(typeof(Animator))]
public class RootMotionMovement : MonoBehaviour
{
    [SerializeField] private Transform _cameraTransform;
    
    private Animator _animator;
    private static readonly int SpeedHash = Animator.StringToHash("Speed");
    private static readonly int DirectionHash = Animator.StringToHash("Direction");
    
    private void Awake()
    {
        _animator = GetComponent<Animator>();
    }
    
    private void Update()
    {
        float vertical = Input.GetAxis("Vertical");
        float horizontal = Input.GetAxis("Horizontal");
        
        // Feed input to Animator — blend tree handles animation selection
        // The animation itself moves the character via root motion
        _animator.SetFloat(SpeedHash, vertical, 0.1f, Time.deltaTime);
        _animator.SetFloat(DirectionHash, horizontal, 0.1f, Time.deltaTime);
    }
    
    private void OnAnimatorMove()
    {
        // Called by Unity when root motion is applied
        // Override to add custom behavior (e.g., apply to Rigidbody instead)
        transform.position += _animator.deltaPosition;
        transform.rotation *= _animator.deltaRotation;
    }
}
```

**Root motion = animation controls speed.** Walk animation moves the character at walk speed. Run animation moves at run speed. You don't set speed in code — the animation data drives it. This is how Dark Souls handles movement.

---

## 8. Common Gotchas

### Transform.Translate — Almost Never Use for Player Movement

```csharp
// DON'T do this for player movement
transform.Translate(direction * speed * Time.deltaTime);
// or
transform.position += direction * speed * Time.deltaTime;
```

This teleports the object. It ignores colliders — the player walks through walls. The only valid use: moving things that don't need collision (UI elements, decorations, particle-like effects).

### "My Character Jitters"

|Symptom|Cause|Fix|
|---|---|---|
|Jittery when moving|Rigidbody without interpolation|Set `Interpolation: Interpolate`|
|Jittery camera|Camera follows in `Update`, player moves in `FixedUpdate`|Move camera in `LateUpdate`, or use Cinemachine|
|Vibrating against walls|Setting velocity into walls every frame|Check collision normals, or use CharacterController|
|Shaking on slopes|Rigidbody sliding on angled surfaces|Increase friction, or use CharacterController|

### "My Character Slides Down Slopes"

CharacterController handles this with `Slope Limit`. Rigidbody doesn't. For Rigidbody on slopes:

```csharp
// Quick fix: add a physics material with high friction to the collider
// Or: zero out velocity when grounded and not moving
if (_isGrounded && _moveInput.magnitude < 0.1f)
{
    _rb.velocity = new Vector3(0f, _rb.velocity.y, 0f);
}
```

### "My Character Can Jump Infinitely"

You're not checking `isGrounded` before allowing jump. Every jump implementation needs a ground check. See Section 3.

### Diagonal Speed Boost

Moving forward + right simultaneously gives √2 (1.414×) speed. Always normalize or clamp your input direction:

```csharp
// 2D
Vector2 input = new Vector2(horizontal, vertical).normalized;

// 3D
Vector3 move = Vector3.ClampMagnitude(moveDir, 1f);
```

### "My Rigidbody Character Walks Through Walls at High Speed"

Set Collision Detection to `Continuous` on the Rigidbody. Default `Discrete` can miss thin colliders at high speeds.

---

## 9. Approach Comparison Summary

|Approach|Move In|Gravity|Collisions|Ground Check|Best For|
|---|---|---|---|---|---|
|`transform.Translate`|Update|Manual|None|Manual|Non-physics objects only|
|`CharacterController`|Update|Manual|Built-in slide|`isGrounded`|FPS, 3rd person, precise control|
|`Rigidbody` + velocity|FixedUpdate|Auto|Auto|Manual|Platformers, physics games|
|`Rigidbody` + AddForce|FixedUpdate|Auto|Auto|Manual|Momentum-heavy, vehicles|
|`Rigidbody` + MovePosition|FixedUpdate|Manual|Auto|Manual|Top-down, kinematic NPCs|
|`NavMeshAgent`|Auto|Auto|Pathfinding|Built-in|AI, NPCs|

---

## 10. Godot → Unity Translation Table

|Godot|Unity Equivalent|
|---|---|
|`CharacterBody3D`|`CharacterController` or `Rigidbody` + your code|
|`CharacterBody2D`|`Rigidbody2D` + your code|
|`move_and_slide(velocity)`|`CharacterController.Move()` or set `rb.velocity`|
|`move_and_collide(velocity)`|`Rigidbody.MovePosition()`|
|`is_on_floor()`|`cc.isGrounded` or `Physics.CheckSphere()`|
|`is_on_wall()`|`cc.collisionFlags.HasFlag(CollisionFlags.Sides)`|
|`is_on_ceiling()`|`cc.collisionFlags.HasFlag(CollisionFlags.Above)`|
|`velocity` property|Track your own `Vector3 velocity`|
|`get_slide_collision()`|`OnControllerColliderHit` callback|
|`_physics_process(delta)`|`FixedUpdate()`|
|`_process(delta)`|`Update()`|
|Floor snap / stop on slope|`CharacterController.Slope Limit` + `Step Offset`|

---

## Key Takeaways

1. **Start with `CharacterController` for 3D** — it handles slopes, steps, and sliding. Add Rigidbody later if you need physics interactions.
2. **Use `Rigidbody2D` + direct velocity for 2D platformers** — the most common, battle-tested approach.
3. **Input in `Update`, physics in `FixedUpdate`** — always. No exceptions.
4. **Normalize diagonal input** — or your character moves 41% faster diagonally.
5. **Enable Interpolation on the player's Rigidbody** — the #1 fix for jittery movement.
6. **Ground check is your responsibility** — Unity doesn't give you `is_on_floor()` for free (except CharacterController's `isGrounded`).
7. **Camera in `LateUpdate`** — after the player has moved.
8. **Don't use `transform.Translate` for player movement** — it ignores collisions.