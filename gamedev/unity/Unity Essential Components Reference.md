# Unity Built-in Components Reference

> Quick reference for Unity's essential built-in components. Know what exists so you don't reinvent it.

---

## Transform & Hierarchy

|Component|What It Does|When to Use|
|---|---|---|
|**Transform**|Position, rotation, scale + parent-child hierarchy|Always present. Can't remove.|
|**RectTransform**|Transform for UI elements (anchors, pivots, sizing)|Auto-added to UI objects.|

---

## Physics 3D

|Component|What It Does|When to Use|
|---|---|---|
|**Rigidbody**|Enables physics simulation (gravity, forces, collisions)|Anything that moves via physics. Required for collision _events_.|
|**BoxCollider**|Box-shaped collision detection|Crates, walls, rectangular objects|
|**SphereCollider**|Sphere-shaped collision|Balls, pickups, simple characters|
|**CapsuleCollider**|Capsule-shaped collision|Characters, humanoids (standing pill shape)|
|**MeshCollider**|Collision matching exact mesh shape|Complex static geometry. **Expensive** — avoid on moving objects.|
|**CharacterController**|Built-in character movement (no physics)|Simple FPS/3rd-person movement. Alternative to Rigidbody.|
|**Joints** (Fixed, Hinge, Spring, etc.)|Connect Rigidbodies with constraints|Doors, chains, ragdolls, vehicles|

### Rigidbody Settings Quick Reference

|Setting|Use Case|
|---|---|
|`isKinematic = true`|Move via transform, still detect collisions (platforms, elevators)|
|`useGravity = false`|Space games, flying objects|
|`constraints`|Lock position/rotation axes (2D in 3D, prevent tipping)|
|`interpolation`|Smooth visual jitter (set to Interpolate for player)|
|`collisionDetection = Continuous`|Fast-moving objects (bullets) to prevent tunneling|

### Collider as Trigger

```csharp
// Set isTrigger = true on collider
// Then use these callbacks:
void OnTriggerEnter(Collider other) { }
void OnTriggerStay(Collider other) { }
void OnTriggerExit(Collider other) { }
```

Use for: Pickups, zones, detection areas (no physical blocking).

---

## Physics 2D

|Component|What It Does|When to Use|
|---|---|---|
|**Rigidbody2D**|2D physics simulation|2D games with physics|
|**BoxCollider2D**|2D box collision|Platforms, crates|
|**CircleCollider2D**|2D circle collision|Balls, coins|
|**CapsuleCollider2D**|2D capsule collision|2D characters|
|**PolygonCollider2D**|Custom 2D shape|Complex shapes (auto-generates from sprite)|
|**CompositeCollider2D**|Merges multiple colliders|Tilemaps, level geometry optimization|

### Rigidbody2D Body Types

|Type|Use Case|
|---|---|
|`Dynamic`|Full physics (player, enemies, physics objects)|
|`Kinematic`|Move via code, detect collisions (platforms, bullets)|
|`Static`|Never moves (ground, walls)|

---

## Rendering 3D

|Component|What It Does|When to Use|
|---|---|---|
|**MeshFilter**|Holds the mesh data|Required for MeshRenderer|
|**MeshRenderer**|Renders the mesh with materials|Any 3D object you can see|
|**SkinnedMeshRenderer**|Renders deformable mesh (bones)|Animated characters, cloth|
|**LineRenderer**|Draws lines in world space|Lasers, ropes, trajectories|
|**TrailRenderer**|Draws fading trail behind object|Projectiles, motion trails|
|**ParticleSystemRenderer**|Renders particle system|Auto-added with ParticleSystem|

### Material Quick Reference

```csharp
// Get/set material (creates instance — won't affect other objects)
renderer.material.color = Color.red;

// Shared material (affects all objects using it)
renderer.sharedMaterial.color = Color.red;  // Careful!

// Multiple materials
renderer.materials[0].color = Color.red;
```

---

## Rendering 2D

|Component|What It Does|When to Use|
|---|---|---|
|**SpriteRenderer**|Renders a 2D sprite|Any 2D visual|
|**SpriteMask**|Masks other sprites|Reveal/hide effects, portals|
|**TilemapRenderer**|Renders tilemap|2D level geometry|

### SpriteRenderer Quick Reference

```csharp
spriteRenderer.sprite = newSprite;
spriteRenderer.color = Color.red;           // Tint
spriteRenderer.flipX = true;                // Mirror horizontally
spriteRenderer.sortingOrder = 5;            // Draw order
spriteRenderer.sortingLayerName = "Player";
```

---

## Animation

|Component|What It Does|When to Use|
|---|---|---|
|**Animator**|State machine-based animation|Characters, complex animation logic|
|**Animation** (Legacy)|Simple clip playback|Avoid — use Animator instead|

### Animator Quick Reference

```csharp
// Trigger state change
animator.SetTrigger("Jump");
animator.SetBool("IsRunning", true);
animator.SetFloat("Speed", 5.5f);
animator.SetInteger("State", 2);

// Direct play (bypasses state machine)
animator.Play("StateName");

// Get current state info
AnimatorStateInfo info = animator.GetCurrentAnimatorStateInfo(0);
if (info.IsName("Idle")) { }
```

---

## Audio

|Component|What It Does|When to Use|
|---|---|---|
|**AudioSource**|Plays audio clips|Any sound emitter|
|**AudioListener**|Receives audio (the "ears")|One per scene, usually on Camera|
|**AudioReverbZone**|Applies reverb in area|Caves, large rooms|

### AudioSource Quick Reference

```csharp
// One-shot (overlapping OK)
audioSource.PlayOneShot(clip, volume);

// Standard play (restarts if called again)
audioSource.clip = clip;
audioSource.Play();

// 3D sound at position (no AudioSource needed)
AudioSource.PlayClipAtPoint(clip, position, volume);

// Settings
audioSource.loop = true;
audioSource.spatialBlend = 1f;  // 0 = 2D, 1 = 3D
audioSource.pitch = 1.2f;
```

---

## UI (Canvas System)

|Component|What It Does|When to Use|
|---|---|---|
|**Canvas**|Container for all UI|Root of UI hierarchy|
|**CanvasScaler**|Handles resolution scaling|On Canvas — set reference resolution|
|**GraphicRaycaster**|Enables UI interaction|On Canvas — required for clicks|
|**Image**|Displays sprite in UI|Icons, backgrounds, health bars|
|**RawImage**|Displays texture (no sprite)|Render textures, videos|
|**Text (Legacy)**|Displays text|Use TextMeshPro instead|
|**TextMeshPro - Text**|High-quality text|All text in UI|
|**Button**|Clickable element|Buttons|
|**Toggle**|Checkbox / radio button|Options, selections|
|**Slider**|Draggable value|Volume, progress|
|**Scrollbar**|Scroll control|Scroll views|
|**Dropdown**|Selection list|Options menus|
|**InputField**|Text input|Name entry, chat|
|**ScrollRect**|Scrollable container|Inventory, lists|
|**Mask**|Clips children to shape|Scroll areas, circular frames|
|**LayoutGroup** (Horizontal, Vertical, Grid)|Auto-arranges children|Lists, grids, menus|
|**ContentSizeFitter**|Auto-sizes to content|Dynamic text, lists|

### Canvas Render Modes

|Mode|Use Case|
|---|---|
|`Screen Space - Overlay`|Standard UI (always on top)|
|`Screen Space - Camera`|UI with depth (particles in front/behind)|
|`World Space`|In-world UI (health bars over heads, signs)|

### Button Quick Reference

```csharp
// Via Inspector: Drag object + select method in OnClick()

// Via code:
button.onClick.AddListener(() => DoSomething());
button.onClick.AddListener(DoSomething);  // Method reference

// Remove
button.onClick.RemoveAllListeners();
```

---

## Camera

|Component|What It Does|When to Use|
|---|---|---|
|**Camera**|Renders scene to screen|At least one per scene|

### Camera Quick Reference

```csharp
// Main camera shortcut
Camera.main.transform.position;

// World to screen position (for UI following objects)
Vector3 screenPos = Camera.main.WorldToScreenPoint(worldPosition);

// Screen to world (for mouse picking)
Ray ray = Camera.main.ScreenPointToRay(Input.mousePosition);

// Viewport (0-1 range)
Vector3 viewportPos = Camera.main.WorldToViewportPoint(worldPosition);
```

### Camera Settings

|Setting|Use Case|
|---|---|
|`clearFlags`|What to show where nothing renders (Skybox, Solid Color)|
|`cullingMask`|Which layers this camera renders|
|`orthographic`|2D games, isometric views|
|`orthographicSize`|Zoom level for orthographic|
|`fieldOfView`|FOV for perspective (60-90 typical)|
|`depth`|Render order when multiple cameras|

---

## Light

|Component|What It Does|When to Use|
|---|---|---|
|**Light**|Illuminates scene|Various types below|

### Light Types

|Type|Use Case|
|---|---|
|`Directional`|Sun, moon (infinite parallel rays)|
|`Point`|Bulbs, torches (radiates from point)|
|`Spot`|Flashlights, stage lights (cone)|
|`Area`|Soft rectangular light (baked only)|

---

## Navigation (AI Pathfinding)

|Component|What It Does|When to Use|
|---|---|---|
|**NavMeshAgent**|Pathfinding movement on NavMesh|AI characters that navigate|
|**NavMeshObstacle**|Dynamic obstacle avoidance|Moving obstacles AI should avoid|
|**OffMeshLink**|Connect disconnected NavMesh areas|Jumps, ladders, teleports|

### NavMeshAgent Quick Reference

```csharp
// Move to destination
agent.SetDestination(targetPosition);

// Check if arrived
if (!agent.pathPending && agent.remainingDistance <= agent.stoppingDistance)
{
    // Arrived
}

// Stop movement
agent.isStopped = true;
agent.ResetPath();

// Settings
agent.speed = 5f;
agent.stoppingDistance = 1f;
agent.angularSpeed = 120f;
```

---

## Input (New Input System)

|Component|What It Does|When to Use|
|---|---|---|
|**PlayerInput**|Connects Input Actions to GameObject|Player-controlled objects|

### Input Quick Reference (New System)

```csharp
// Via PlayerInput component callbacks
public void OnMove(InputValue value)
{
    Vector2 input = value.Get<Vector2>();
}

public void OnJump(InputValue value)
{
    if (value.isPressed) Jump();
}

// Direct reading
var action = inputActions.Player.Move;
Vector2 move = action.ReadValue<Vector2>();
```

### Input Quick Reference (Legacy)

```csharp
// Axes
float h = Input.GetAxis("Horizontal");        // Smoothed
float hRaw = Input.GetAxisRaw("Horizontal");  // -1, 0, or 1

// Buttons
if (Input.GetButtonDown("Jump")) { }   // Frame pressed
if (Input.GetButton("Fire1")) { }      // Held
if (Input.GetButtonUp("Jump")) { }     // Frame released

// Keys
if (Input.GetKeyDown(KeyCode.Space)) { }

// Mouse
Vector3 mousePos = Input.mousePosition;
if (Input.GetMouseButtonDown(0)) { }   // Left click
```

---

## Miscellaneous

|Component|What It Does|When to Use|
|---|---|---|
|**ParticleSystem**|Particle effects|Fire, smoke, sparks, magic|
|**VideoPlayer**|Plays video|Cutscenes, in-world screens|
|**Terrain**|Large-scale terrain|Open worlds, landscapes|
|**WindZone**|Affects particles and trees|Environmental wind|
|**LODGroup**|Level-of-detail switching|Optimization for distant objects|
|**ReflectionProbe**|Captures reflections|Shiny surfaces, mirrors|
|**LightProbeGroup**|Baked lighting for dynamic objects|Moving objects in baked scenes|

---

## Common Component Combinations

### Basic 3D Object (Static)

```
GameObject
├── MeshFilter
├── MeshRenderer
└── Collider (Box/Mesh)
```

### Physics Object

```
GameObject
├── MeshFilter
├── MeshRenderer
├── Collider
└── Rigidbody
```

### Character (Physics-based)

```
GameObject
├── Mesh/SkinnedMeshRenderer
├── CapsuleCollider
├── Rigidbody (constraints: freeze rotation X, Z)
├── Animator
└── YourController.cs
```

### Character (CharacterController)

```
GameObject
├── Mesh/SkinnedMeshRenderer
├── CharacterController
├── Animator
└── YourController.cs
```

### 2D Character

```
GameObject
├── SpriteRenderer
├── CapsuleCollider2D / BoxCollider2D
├── Rigidbody2D
├── Animator
└── YourController.cs
```

### UI Button

```
Button (GameObject)
├── RectTransform
├── CanvasRenderer
├── Image (background)
├── Button (component)
└── Child: Text (TextMeshPro)
```

### Audio Emitter

```
GameObject
├── AudioSource
└── TriggerZone.cs (to play on enter, etc.)
```

### AI Enemy

```
GameObject
├── Mesh + Renderer
├── CapsuleCollider
├── NavMeshAgent
├── Animator
├── Health.cs
└── EnemyAI.cs
```

---

## Quick Decision Tree

```
Need physics simulation?
├── YES → Rigidbody + Collider
└── NO → Just Collider (for triggers) or nothing

Need character movement?
├── Physics-based (slopes, pushing) → Rigidbody + Collider + your code
├── Simple FPS/3rd person → CharacterController
└── AI pathfinding → NavMeshAgent

Rendering something?
├── 3D mesh → MeshFilter + MeshRenderer
├── 3D animated → SkinnedMeshRenderer + Animator
├── 2D sprite → SpriteRenderer
└── UI element → Image / TextMeshPro under Canvas

Playing audio?
├── Background music → AudioSource (loop, 2D)
├── 3D sound → AudioSource (3D spatial blend)
└── One-shot effect → audioSource.PlayOneShot() or PlayClipAtPoint()

UI needed?
└── Create Canvas → Add UI elements as children
```

---

## Don't Forget

- **Rigidbody required for collision events** (`OnCollisionEnter`) even if kinematic
- **AudioListener** — only one active per scene (usually on main camera)
- **EventSystem** — required for UI interaction (auto-created with Canvas)
- **TextMeshPro** — always use over legacy Text component
- **Layers & Tags** — set up for collision filtering and object identification**