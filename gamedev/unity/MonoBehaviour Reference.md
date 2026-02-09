# MonoBehaviour

> [!quote] The Core Insight MonoBehaviour is the bridge between your C# code and Unity's engine. It's how your code becomes a living part of the game.

---

## What is MonoBehaviour?

In Godot, you write a script that `extends Node` and attach it to a node. That script becomes part of the node.

In Unity, the equivalent is **MonoBehaviour**. It's a base class that turns your C# class into something that can:

- Attach to GameObjects
- Respond to Unity's lifecycle (Awake, Update, etc.)
- Be configured in the Inspector
- Interact with the engine

```csharp
public class Enemy : MonoBehaviour  // This can now attach to a GameObject
{
}
```

Without inheriting from MonoBehaviour, your class is just a plain C# class — Unity doesn't know it exists.

---

## Creating MonoBehaviours: The Rules

### You Cannot Use `new`

This is the most important rule coming from other languages:

```csharp
// WRONG — will create a broken object
Enemy enemy = new Enemy();

// This creates a C# object but Unity doesn't know about it.
// It won't be attached to any GameObject.
// Lifecycle methods won't be called.
// It will cause errors.
```

**Why?** MonoBehaviours must be attached to GameObjects. Unity manages their creation as part of the GameObject system. Using `new` bypasses this.

### Correct Ways to Create MonoBehaviours

**1. Add component to existing GameObject:**

```csharp
Enemy enemy = gameObject.AddComponent<Enemy>();
// or
Enemy enemy = someOtherGameObject.AddComponent<Enemy>();
```

**2. Instantiate a prefab that has the component:**

```csharp
// If enemyPrefab has an Enemy component on it:
GameObject obj = Instantiate(enemyPrefab);
Enemy enemy = obj.GetComponent<Enemy>();

// Or if you reference the component directly:
[SerializeField] private Enemy _enemyPrefab;
Enemy enemy = Instantiate(_enemyPrefab);  // Returns Enemy, not GameObject
```

**3. Already exists in the scene:**

```csharp
Enemy enemy = FindObjectOfType<Enemy>();
```

### You Cannot Use Constructors

Since you can't use `new`, constructors are useless for initialization:

```csharp
public class Enemy : MonoBehaviour
{
    // WRONG — don't do this
    public Enemy()
    {
        // This may or may not run
        // Unity may create instances in unexpected ways (editor, serialization)
        // Behavior is undefined
    }
    
    // RIGHT — use Awake instead
    private void Awake()
    {
        // This is your "constructor" in Unity
    }
}
```

---

## What MonoBehaviour Gives You

When you inherit from MonoBehaviour, you get access to:

### Built-in References

```csharp
gameObject     // The GameObject this component is attached to
transform      // Shortcut to gameObject.transform (position, rotation, scale)
enabled        // Is this component enabled? (affects Update, etc.)
```

### Methods You Can Call

```csharp
// Get other components on same GameObject
GetComponent<Rigidbody>()
GetComponents<Collider>()
TryGetComponent<Health>(out var health)

// Get components on children/parent
GetComponentInChildren<Renderer>()
GetComponentInParent<Canvas>()
GetComponentsInChildren<Enemy>()

// Instantiate prefabs
Instantiate(prefab)
Instantiate(prefab, position, rotation)

// Destroy things
Destroy(gameObject)       // Destroy at end of frame
Destroy(gameObject, 2f)   // Destroy after 2 seconds
Destroy(this)             // Destroy just this component, not the GameObject

// Coroutines
StartCoroutine(MyCoroutine())
StopCoroutine(coroutineReference)
StopAllCoroutines()

// Invoke (delayed method calls)
Invoke("MethodName", delay)
InvokeRepeating("MethodName", delay, repeatRate)
CancelInvoke()
```

### Lifecycle Methods (Unity Calls These)

These are methods Unity automatically calls at specific moments. You don't call them yourself — you define them, Unity calls them.

```csharp
private void Awake() { }       // Called when object is created
private void Start() { }       // Called before first frame
private void Update() { }      // Called every frame
// ... and many more
```

---

## The Lifecycle: When Unity Calls Your Methods

### The Big Picture

```
OBJECT CREATED (Instantiate or scene load)
    │
    ▼
  Awake()          ← Always runs, even if disabled
    │
    ▼
  OnEnable()       ← Only if component is enabled
    │
    ▼
  Start()          ← First frame, only once ever
    │
    ▼
┌─────────────────── GAME LOOP (repeats) ───────────────────┐
│                                                            │
│   FixedUpdate()  ← Physics tick (may run 0, 1, or N times) │
│        │                                                   │
│        ▼                                                   │
│   Update()       ← Once per frame                          │
│        │                                                   │
│        ▼                                                   │
│   LateUpdate()   ← After all Updates                       │
│                                                            │
└────────────────────────────────────────────────────────────┘
    │
    ▼ (when disabled or destroyed)
  OnDisable()      ← Component disabled or object deactivated
    │
    ▼
  OnDestroy()      ← Object being destroyed
```

### Initialization: Awake → OnEnable → Start

**Awake** — Called once when the object is created (via Instantiate or scene load). Runs even if the component is disabled.

```csharp
private void Awake()
{
    // Initialize yourself. Get your own components.
    _rb = GetComponent<Rigidbody>();
    _health = GetComponent<Health>();
    
    // Set up internal state
    _inventory = new List<Item>();
    _stateMachine = new StateMachine();
}
```

**When to use Awake:**

- Getting components on the same GameObject
- Initializing internal data structures
- Setting up things that don't depend on other objects

**When NOT to use Awake:**

- Finding other GameObjects (they might not exist yet)
- Accessing singletons that are set up in their own Awake (order is undefined)

---

**OnEnable** — Called each time the component is enabled. This includes right after Awake (if the component starts enabled) and whenever you set `enabled = true` or reactivate the GameObject.

```csharp
private void OnEnable()
{
    // Subscribe to events
    GameEvents.OnPlayerDied += HandlePlayerDeath;
    _health.OnDepleted += Die;
    
    // Register with systems
    EnemyManager.Register(this);
}
```

**When to use OnEnable:**

- Subscribing to events
- Registering with manager systems
- Anything that should happen each time the object becomes active

---

**Start** — Called once, on the first frame the object is active. All Awakes in the scene have already run.

```csharp
private void Start()
{
    // Safe to find other objects now
    _player = FindObjectOfType<Player>();
    _target = _player.transform;
    
    // Initial state that depends on other objects
    _homePosition = transform.position;
}
```

**When to use Start:**

- Finding references to other GameObjects
- Setup that depends on other objects being initialized
- Initial state that needs the world to be ready

This is used like a two step step, that is done in one in godot's `_ready`. Since Start is called after all `Awake` is called, meaning every object's internal state is ready

---

### The Critical Difference from Godot

In Godot, `_ready()` runs every time a node enters the tree. Reparent a node? `_ready()` runs again.

**In Unity, `Start()` runs ONCE. Ever.** If you disable and re-enable a GameObject, Start does NOT run again. If you reparent an object, Start does NOT run again.

For "each time activated" logic, use `OnEnable()`.

---

### Game Loop: FixedUpdate → Update → LateUpdate

**FixedUpdate** — Runs on a fixed timer, independent of frame rate. Default is 50 times per second (every 0.02 seconds). May run 0, 1, or multiple times per frame depending on how fast your game is running.

```csharp
private void FixedUpdate()
{
    // Physics operations
    _rb.AddForce(Vector3.up * jumpForce);
    _rb.MovePosition(transform.position + velocity * Time.fixedDeltaTime);
}
```

**When to use FixedUpdate:**

- Anything involving Rigidbody (forces, velocity, MovePosition)
- Physics-based movement
- Anything that needs consistent timing regardless of frame rate

**Time value:** Use `Time.fixedDeltaTime` (constant, usually 0.02)

---

**Update** — Runs once per frame. Frame rate dependent — runs more often on fast computers, less often on slow ones.

```csharp
private void Update()
{
    // Input (must check every frame or you'll miss button presses)
    if (Input.GetKeyDown(KeyCode.Space))
        Jump();
    
    // Visual updates
    _animator.SetFloat("Speed", _currentSpeed);
    
    // Non-physics movement
    transform.Rotate(Vector3.up, rotationSpeed * Time.deltaTime);
    
    // Game logic
    _stateTimer -= Time.deltaTime;
    if (_stateTimer <= 0)
        ChangeState();
}
```

**When to use Update:**

- Input detection
- Non-physics movement (transform-based)
- Visual updates (animation parameters)
- Timers and game logic

**Time value:** Use `Time.deltaTime` (varies each frame)

---

**LateUpdate** — Runs once per frame, after ALL Update methods have finished.

```csharp
private void LateUpdate()
{
    // Camera follow — player has already moved in their Update
    transform.position = _target.position + _offset;
    
    // Look at target — target has already rotated
    transform.LookAt(_target);
}
```

**When to use LateUpdate:**

- Camera follow (after the thing you're following has moved)
- Any logic that must run after other objects have updated
- Procedural animation that responds to final positions

---

### Decommission: OnDisable → OnDestroy

**OnDisable** — Called each time the component is disabled. This includes when `enabled = false`, when the GameObject is deactivated, and right before OnDestroy.

```csharp
private void OnDisable()
{
    // Unsubscribe from events (CRITICAL — prevents memory leaks)
    GameEvents.OnPlayerDied -= HandlePlayerDeath;
    _health.OnDepleted -= Die;
    
    // Unregister from systems
    EnemyManager.Unregister(this);
}
```

**Why unsubscribing matters:**

C# events hold strong references. If object A subscribes to object B's event:

- B keeps a reference to A
- If A is destroyed but didn't unsubscribe, B still holds a dead reference
- Next time B fires the event → NullReferenceException
- Also: A can't be garbage collected → memory leak

**Rule:** Always unsubscribe in OnDisable what you subscribed in OnEnable.

---

**OnDestroy** — Called once when the object is being destroyed (via `Destroy()` or scene unload).

```csharp
private void OnDestroy()
{
    // Final cleanup
    SaveProgress();
    
    // Release unmanaged resources
    _nativeArray.Dispose();
}
```

**When to use OnDestroy:**

- Final cleanup that should only happen when truly destroyed
- Saving state
- Releasing resources that OnDisable didn't handle

---

## Other Useful Lifecycle Methods

### Physics Callbacks

```csharp
// Collision (physics colliders touching)
void OnCollisionEnter(Collision collision) { }
void OnCollisionStay(Collision collision) { }
void OnCollisionExit(Collision collision) { }

// Triggers (collider with isTrigger = true)
void OnTriggerEnter(Collider other) { }
void OnTriggerStay(Collider other) { }
void OnTriggerExit(Collider other) { }

// 2D versions
void OnCollisionEnter2D(Collision2D collision) { }
void OnTriggerEnter2D(Collider2D other) { }
```

### Visibility

```csharp
void OnBecameVisible() { }   // Any camera can see this renderer
void OnBecameInvisible() { } // No camera can see this renderer
```

### Application State

```csharp
void OnApplicationFocus(bool hasFocus) { }  // Window focus changed
void OnApplicationPause(bool pauseStatus) { } // Mobile: app backgrounded
void OnApplicationQuit() { }  // Application closing
```

### Editor Only

```csharp
void OnValidate() { }     // Inspector value changed (editor only)
void OnDrawGizmos() { }   // Draw debug visuals (editor only)
void Reset() { }          // Component added or Reset clicked (editor only)
```

---

## Coroutines: Spreading Work Over Time

Coroutines let you write code that executes over multiple frames without blocking.

```csharp
private IEnumerator FadeOut()
{
    float alpha = 1f;
    
    while (alpha > 0)
    {
        alpha -= Time.deltaTime;
        _renderer.material.color = new Color(1, 1, 1, alpha);
        yield return null;  // Wait until next frame
    }
    
    Destroy(gameObject);
}

// Start the coroutine
StartCoroutine(FadeOut());
```

### Common Yield Instructions

```csharp
yield return null;                          // Wait one frame
yield return new WaitForSeconds(2f);        // Wait 2 seconds
yield return new WaitForSecondsRealtime(2f); // Wait 2 seconds (ignores Time.timeScale)
yield return new WaitForFixedUpdate();      // Wait until next FixedUpdate
yield return new WaitForEndOfFrame();       // Wait until rendering is done
yield return new WaitUntil(() => _isReady); // Wait until condition is true
yield return new WaitWhile(() => _isBusy);  // Wait while condition is true
yield return StartCoroutine(Other());       // Wait for another coroutine
yield return asyncOperation;                // Wait for async operation (scene load, etc.)
```

### Stopping Coroutines

```csharp
// Store reference to stop later
Coroutine routine = StartCoroutine(MyRoutine());
StopCoroutine(routine);

// Stop all coroutines on this MonoBehaviour
StopAllCoroutines();

// Coroutines also stop when:
// - The MonoBehaviour is disabled
// - The GameObject is deactivated
// - The object is destroyed
```

---

## Enabling and Disabling

### Component vs GameObject

```csharp
// Disable component — only this component stops updating
GetComponent<Enemy>().enabled = false;

// Deactivate GameObject — entire object and children stop
gameObject.SetActive(false);
```

### What `enabled = false` Does

When a MonoBehaviour is disabled:

- Update, FixedUpdate, LateUpdate **stop running**
- Coroutines **stop**
- OnDisable is called

What still works:

- The component still exists
- You can still call its methods manually
- GetComponent still finds it
- OnCollision/OnTrigger **still work** (these are on the Collider, not the script)

### What `SetActive(false)` Does

When a GameObject is deactivated:

- All components stop (as if all were disabled)
- All children are also deactivated
- OnDisable is called on all components
- The object becomes invisible and non-interacting

---

## The Standard Pattern

Putting it all together:

```csharp
public class Enemy : MonoBehaviour
{
    // Serialized fields (configured in Inspector)
    [SerializeField] private float _speed = 5f;
    [SerializeField] private int _maxHealth = 100;
    
    // Cached component references
    private Rigidbody _rb;
    private Health _health;
    private Animator _animator;
    
    // Runtime state
    private Transform _target;
    private bool _isAggro;
    
    // Events
    public event Action OnDeath;
    
    private void Awake()
    {
        // Cache components (self-references)
        _rb = GetComponent<Rigidbody>();
        _health = GetComponent<Health>();
        _animator = GetComponent<Animator>();
    }
    
    private void OnEnable()
    {
        // Subscribe to events
        _health.OnDepleted += Die;
    }
    
    private void OnDisable()
    {
        // Always unsubscribe
        _health.OnDepleted -= Die;
    }
    
    private void Start()
    {
        // Find other objects (world is ready)
        _target = FindObjectOfType<Player>().transform;
    }
    
    private void Update()
    {
        // Input, logic, non-physics updates
        if (_isAggro)
            _animator.SetFloat("Speed", _rb.velocity.magnitude);
    }
    
    private void FixedUpdate()
    {
        // Physics
        if (_isAggro)
            ChaseTarget();
    }
    
    private void Die()
    {
        OnDeath?.Invoke();
        Destroy(gameObject);
    }
}
```

---

## MonoBehaviour vs Plain C# Classes

Not everything should be a MonoBehaviour.

|Use MonoBehaviour|Use Plain C# Class|
|---|---|
|Needs to attach to GameObject|Doesn't need a GameObject|
|Needs lifecycle (Update, etc.)|Just data or logic|
|Needs Inspector configuration|Configured via code|
|Needs coroutines|Can use async/await instead|
|Examples: PlayerController, Enemy, Projectile|Examples: HealthSystem, InventoryData, StateMachine|

See [[Layered Architecture]] for when to use plain C# classes.

---

## Godot Comparison

|Godot|Unity|Difference|
|---|---|---|
|`extends Node`|`: MonoBehaviour`|Same concept|
|`_init()`|Constructor (don't use)|Use Awake instead|
|`_ready()`|`Start()`|Start runs ONCE EVER, not on re-enable|
|`_enter_tree()`|`Awake()` + `OnEnable()`|Awake once, OnEnable each time|
|`_exit_tree()`|`OnDisable()` + `OnDestroy()`|OnDisable each time, OnDestroy once|
|`_process(delta)`|`Update()`|Same|
|`_physics_process(delta)`|`FixedUpdate()`|Same|
|`queue_free()`|`Destroy(gameObject)`|Same|
|`$Child`|`GetComponent<T>()` / `[SerializeField]`|Different access pattern|

---

## Key Takeaways

1. **MonoBehaviour = script that attaches to GameObject**
2. **Never use `new`** — use AddComponent or Instantiate
3. **Never use constructors** — use Awake
4. **Awake** = self-setup (once, even if disabled)
5. **OnEnable/OnDisable** = subscribe/unsubscribe (each time enabled/disabled)
6. **Start** = cross-object setup (once ever, not on re-enable)
7. **Update** = per-frame logic, **FixedUpdate** = physics, **LateUpdate** = after everything
8. **Always unsubscribe events** in OnDisable to prevent memory leaks
9. **Not everything should be a MonoBehaviour** — use plain C# classes for data and systems