# Execution Order

> [!quote] The Core Insight Unity doesn't guarantee script order. If order matters, control it explicitly.

---

## The Problem

By default, the order scripts execute is **undefined**. If ScriptA's `Awake()` depends on ScriptB being initialized first, you have a race condition.

```csharp
// ScriptA
private void Awake()
{
    _manager = FindObjectOfType<GameManager>();
    _manager.Register(this);  // Might be null if GameManager hasn't run Awake yet
}
```

---

## Frame Execution Order

Within a single frame, Unity runs methods in this order:

```
┌─────────────────────────────────────┐
│  1. FixedUpdate (0 to N times)      │  ← Physics timestep catch-up
├─────────────────────────────────────┤
│  2. Update                          │  ← Game logic
├─────────────────────────────────────┤
│  3. LateUpdate                      │  ← Post-update (camera, etc.)
├─────────────────────────────────────┤
│  4. Rendering                       │
└─────────────────────────────────────┘
```

But **within each phase**, script order is undefined unless you control it.

---

## Controlling Script Order

### Option 1: Script Execution Order (Project Settings)

Edit → Project Settings → Script Execution Order

- Negative numbers = earlier
- Default = 0
- Positive numbers = later

```
-100  GameManager
-50   InputManager
0     (default scripts)
100   CameraFollow
```

**Pros:** Visual, no code changes **Cons:** Global setting, doesn't scale well with many scripts

### Option 2: [DefaultExecutionOrder] Attribute

```csharp
[DefaultExecutionOrder(-100)]
public class GameManager : MonoBehaviour
{
    // Runs before default scripts
}

[DefaultExecutionOrder(100)]
public class CameraFollow : MonoBehaviour
{
    // Runs after default scripts
}
```

**Pros:** Lives with the code, self-documenting **Cons:** Can't see all orders at a glance

### Option 3: Explicit Initialization

Don't rely on Awake/Start order. Use explicit initialization:

```csharp
public class GameManager : MonoBehaviour
{
    public static GameManager Instance { get; private set; }
    
    private void Awake()
    {
        Instance = this;
    }
    
    public void Initialize()
    {
        // Called explicitly by a bootstrap
    }
}

public class Bootstrap : MonoBehaviour
{
    [DefaultExecutionOrder(-1000)]
    private void Awake()
    {
        GameManager.Instance.Initialize();
        AudioManager.Instance.Initialize();
        // Explicit, controlled order
    }
}
```

---

## Initialization Order Guarantees

Unity does guarantee:

1. **All `Awake()` calls complete before any `Start()`**
2. **`OnEnable()` is called after `Awake()` but before `Start()`**
3. **Parent `Awake()` before children** (if instantiated together)

Unity does NOT guarantee:

- Order of `Awake()` across different objects
- Order of `Start()` across different objects
- Order of `Update()` across different objects

---

## Common Patterns

### Manager Initialization Chain

```csharp
[DefaultExecutionOrder(-100)]
public class GameBootstrap : MonoBehaviour
{
    [SerializeField] private GameConfig _config;
    
    private void Awake()
    {
        // Initialize in explicit order
        ServiceLocator.Initialize();
        
        var gameManager = new GameManager(_config);
        ServiceLocator.Register(gameManager);
        
        var audioManager = new AudioManager();
        ServiceLocator.Register(audioManager);
        
        // etc.
    }
}
```

### Lazy Initialization

Avoid order issues by initializing on first access:

```csharp
public class GameManager : MonoBehaviour
{
    private static GameManager _instance;
    
    public static GameManager Instance
    {
        get
        {
            if (_instance == null)
            {
                _instance = FindObjectOfType<GameManager>();
                _instance.Initialize();
            }
            return _instance;
        }
    }
    
    private bool _initialized;
    
    private void Initialize()
    {
        if (_initialized) return;
        _initialized = true;
        // Setup...
    }
}
```

### Event-Based Dependencies

Instead of assuming order, wait for notification:

```csharp
public class PlayerController : MonoBehaviour
{
    private void OnEnable()
    {
        GameManager.OnInitialized += Setup;
        
        // In case already initialized
        if (GameManager.IsInitialized)
            Setup();
    }
    
    private void OnDisable()
    {
        GameManager.OnInitialized -= Setup;
    }
    
    private void Setup()
    {
        // Safe to use GameManager now
    }
}
```

---

## Update Order Control

For `Update()` order, same options apply:

```csharp
[DefaultExecutionOrder(100)]  // Runs after default Updates
public class CameraFollow : MonoBehaviour
{
    private void Update()
    {
        // Player has already moved
        transform.position = _target.position + _offset;
    }
}
```

Or use `LateUpdate()` which always runs after all `Update()` calls.

---

## Physics Order

`FixedUpdate()` runs on a separate timestep. Within FixedUpdate:

1. All `FixedUpdate()` calls
2. Physics simulation
3. `OnTriggerXxx`, `OnCollisionXxx` callbacks

Control FixedUpdate order the same way as Update.

---

## Godot Comparison

|Godot|Unity|
|---|---|
|Children `_ready()` before parent|No such guarantee|
|`_process` runs in tree order (mostly)|Update order undefined|
|No built-in execution order control|Script Execution Order / Attribute|

**Key difference:** Godot's tree structure implies some ordering. Unity has no implicit order.

---

## Key Takeaways

1. **Script order is undefined** by default
2. **Use `[DefaultExecutionOrder]`** for critical ordering
3. **All Awakes before all Starts** — guaranteed
4. **Explicit initialization** > relying on Awake order
5. **LateUpdate** for "after everything" logic
6. **When in doubt, don't depend on order** — use events or lazy init