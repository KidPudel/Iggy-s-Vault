# MonoBehaviour Lifecycle

> [!quote] The Core Insight Unity calls your methods at specific moments. Know when, or fight the engine.

---

## What is MonoBehaviour?

MonoBehaviour is the **base class for all Unity scripts** that attach to GameObjects.

```csharp
public class Enemy : MonoBehaviour   // Inherits from MonoBehaviour
{
    // Now this script can attach to a GameObject
}
```

**What it gives you:**

- Can attach to GameObjects as a component
- Access to `gameObject`, `transform`, `enabled`
- Lifecycle methods (`Awake`, `Start`, `Update`, etc.)
- Coroutines (`StartCoroutine`)
- Access to `GetComponent`, `Instantiate`, `Destroy`

**What you can NOT do:**

- Create with `new` — Unity manages instantiation
- Use constructors for initialization — use `Awake` instead

```csharp
// WRONG — never do this
Enemy enemy = new Enemy();

// RIGHT — instantiate a GameObject with the component
Enemy enemy = Instantiate(enemyPrefab);

// Or add to existing GameObject
Enemy enemy = gameObject.AddComponent<Enemy>();
```

### MonoBehaviour vs Plain C# Class

| MonoBehaviour                  | Plain C# Class                        |
| ------------------------------ | ------------------------------------- |
| Attaches to GameObject         | Standalone object                     |
| Unity manages lifetime         | You manage lifetime                   |
| Has lifecycle methods          | No lifecycle methods                  |
| Visible in Inspector           | Not visible (unless `[Serializable]`) |
| Can't use `new`                | Use `new` normally                    |
| Use for: components, behaviors | Use for: data, systems, logic         |

**When to use plain C# classes:** Data containers, systems, state machines, anything that doesn't need to be a component. See [[Layered Architecture in Unity]].

### Godot Equivalent

MonoBehaviour is like a script that `extends Node`:

```gdscript
# Godot
extends Node
func _ready():
    pass
```

```csharp
// Unity
public class MyScript : MonoBehaviour
{
    void Start() { }
}
```

The difference: In Godot you extend specific node types (`CharacterBody3D`, `Sprite2D`). In Unity, you always extend `MonoBehaviour` and add other components separately (composition over inheritance).

---

## The Flow

```
                    ┌─────────────────────────────────────┐
                    │         INITIALIZATION              │
                    ├─────────────────────────────────────┤
                    │  Awake()      ← object created      │
                    │  OnEnable()   ← component enabled   │
                    │  Start()      ← first frame only    │
                    └──────────────────┬──────────────────┘
                                       │
                    ┌──────────────────▼──────────────────┐
                    │           GAME LOOP                 │
                    │         (repeats every frame)       │
                    ├─────────────────────────────────────┤
        ┌──────────►│  FixedUpdate()  ← physics tick      │
        │           │  Update()       ← every frame       │
        │           │  LateUpdate()   ← after all Updates │
        │           └──────────────────┬──────────────────┘
        │                              │
        └──────────────────────────────┘
                                       │
                    ┌──────────────────▼──────────────────┐
                    │          DECOMMISSION               │
                    ├─────────────────────────────────────┤
                    │  OnDisable()   ← component disabled │
                    │  OnDestroy()   ← object destroyed   │
                    └─────────────────────────────────────┘
```

---

## Initialization

### Awake

Called **once** when the object is created, even if the component is disabled.

```csharp
private void Awake()
{
    // Cache component references
    _rb = GetComponent<Rigidbody>();
    _health = GetComponent<Health>();
    
    // Initialize internal state
    _inventory = new List<Item>();
}
```

**Use for:** Self-initialization. Getting your own components. Setting up internal state.

**Don't use for:** Accessing other GameObjects — they might not exist yet.

### OnEnable

Called **each time** the component is enabled (including after Awake on first creation).

```csharp
private void OnEnable()
{
    // Subscribe to events
    GameEvents.OnPlayerDied += HandlePlayerDeath;
}
```

**Use for:** Subscribing to events. Registering with systems.

### Start

Called **once**, on the first frame the object is active, after all Awakes have run.

```csharp
private void Start()
{
    // Safe to access other objects now
    _player = FindObjectOfType<Player>();
    
    // Initial setup that depends on other objects
    _target = _player.transform;
}
```

**Use for:** Setup that depends on other objects existing. Cross-object references.

> [!danger] Start Runs Once, Ever Unlike Godot's `_ready()` which runs each time a node enters the tree, Unity's `Start()` runs **once per component instance lifetime**. Re-enabling an object does NOT re-run Start.

---

## Initialization Order Summary

|Method|When|How Often|Use For|
|---|---|---|---|
|Awake|Object created|Once ever|Self-setup, GetComponent|
|OnEnable|Component enabled|Each enable|Event subscription|
|Start|First frame active|Once ever|Cross-object setup|

**Order guarantee:** All Awakes run before any Start. But order _within_ Awakes is undefined (see [[Execution Order]]).

---

## Game Loop

### FixedUpdate

Called at fixed time intervals (default 50 times/second). **Decoupled from frame rate.**

```csharp
private void FixedUpdate()
{
    // Physics movement
    _rb.AddForce(Vector3.up * jumpForce);
    
    // Consistent simulation
    _velocity += gravity * Time.fixedDeltaTime;
}
```

**Use for:** Physics. Anything that needs deterministic timing.

**Time value:** `Time.fixedDeltaTime` (constant, default 0.02s)

### Update

Called **once per frame**. Frame rate dependent.

```csharp
private void Update()
{
    // Input
    if (Input.GetKeyDown(KeyCode.Space))
        Jump();
    
    // Visual updates
    _animator.SetFloat("Speed", _currentSpeed);
    
    // Non-physics movement
    transform.Rotate(Vector3.up, rotationSpeed * Time.deltaTime);
}
```

**Use for:** Input. Visual updates. Game logic. Most gameplay code.

**Time value:** `Time.deltaTime` (varies per frame)

### LateUpdate

Called **after all Updates** have finished.

```csharp
private void LateUpdate()
{
    // Camera follow (after player has moved)
    transform.position = _target.position + _offset;
}
```

**Use for:** Camera follow. Anything that must happen after other objects update.

---

## Game Loop Summary

|Method|Timing|Use For|
|---|---|---|
|FixedUpdate|Fixed interval (physics clock)|Physics, deterministic simulation|
|Update|Once per frame|Input, game logic, visuals|
|LateUpdate|After all Updates|Camera, post-processing logic|

```
Frame timeline:
├── FixedUpdate (0 to N times, catching up to real time)
├── Update (once)
├── LateUpdate (once)
└── Render
```

FixedUpdate may run multiple times per frame (or zero) depending on frame rate vs fixed timestep.

---

## Decommission

### OnDisable

Called **each time** the component is disabled or the GameObject is deactivated.

```csharp
private void OnDisable()
{
    // Unsubscribe from events (prevent memory leaks!)
    GameEvents.OnPlayerDied -= HandlePlayerDeath;
}
```

**Use for:** Unsubscribing from events. Cleanup that should happen on disable.

> [!danger] Always Unsubscribe Events C# events hold strong references. If you subscribe in OnEnable but don't unsubscribe in OnDisable, you get memory leaks and null reference errors when the subscriber is destroyed.

### OnDestroy

Called **once** when the object is destroyed.

```csharp
private void OnDestroy()
{
    // Final cleanup
    SaveProgress();
    
    // Unsubscribe (if not done in OnDisable)
    SomeExternalSystem.Unregister(this);
}
```

**Use for:** Final cleanup. Saving. Unregistering from external systems.

---

## Common Patterns

### The Standard Setup

```csharp
public class Enemy : MonoBehaviour
{
    // Serialized fields
    [SerializeField] private float _speed = 5f;
    
    // Cached references
    private Rigidbody _rb;
    private Health _health;
    
    // Events
    public event Action OnDeath;
    
    private void Awake()
    {
        // Self-references
        _rb = GetComponent<Rigidbody>();
        _health = GetComponent<Health>();
    }
    
    private void OnEnable()
    {
        // Subscribe
        _health.OnDepleted += Die;
    }
    
    private void OnDisable()
    {
        // Unsubscribe
        _health.OnDepleted -= Die;
    }
    
    private void Update()
    {
        // Game logic
        Move();
    }
    
    private void Die()
    {
        OnDeath?.Invoke();
        Destroy(gameObject);
    }
}
```

### Conditional Updates

Don't run logic every frame if you don't need to:

```csharp
private void Update()
{
    if (!_isActive) return;
    
    // Actual logic
}
```

Or disable the component entirely:

```csharp
enabled = false;  // Stops Update/FixedUpdate/LateUpdate
```

---

## Godot Comparison

|Godot|Unity|Key Difference|
|---|---|---|
|`_init()`|Constructor (avoid)|Use Awake instead|
|`_enter_tree()`|Awake|Awake runs once ever, not on re-parent|
|`_ready()`|Start|Start runs once ever, not on re-enable|
|`_process(delta)`|Update|Same concept|
|`_physics_process(delta)`|FixedUpdate|Same concept|
|`_exit_tree()`|OnDestroy|OnDestroy runs once on destroy|
|—|OnEnable/OnDisable|No direct Godot equivalent|

**Critical difference:** Godot's `_ready()` runs each time a node enters the tree. Unity's `Start()` runs once per lifetime. For "each time enabled" logic, use `OnEnable()`.

---

## Key Takeaways

1. **Awake** = self-setup (once)
2. **Start** = cross-object setup (once, after all Awakes)
3. **OnEnable/OnDisable** = subscribe/unsubscribe (each time)
4. **Update** = per-frame logic
5. **FixedUpdate** = physics (fixed timestep)
6. **LateUpdate** = after everything else (camera)
7. **Always unsubscribe** events in OnDisable

See [[Execution Order]] for controlling which scripts run first.