# Dependency Resolution in Unity

> [!quote] The Core Problem
> Your class needs access to a service (AudioManager, HealthSystem). How does it get that reference without tight coupling?

---

# The Problem

Objects need to access services and managers: "I need X to do my job." This is **one-to-one access** — getting a reference to something that exists.

**This document covers service access patterns.** For event-driven communication, see [[Communication Patterns in Unity]].

---

# The Evolution of Dependency Management

Based on [PracticAPI's tutorial](https://www.youtube.com/watch?v=xYRFwFXNskc), here's the maturity model from chaos to industry standard:

| Stage | Technique                                | Verdict              |
| ----- | ---------------------------------------- | -------------------- |
| 1     | `GameObject.Find()`, `FindObjectOfType()`| Avoid — slow, fragile |
| 2     | `[SerializeField]` in Inspector          | Good foundation      |
| 3     | `public static Instance`                 | Trap at scale        |
| 4     | Service Locator                          | Professional         |
| 5     | Dependency Injection                     | Industry standard    |

Goal: Maximize **decoupling**, **scalability**, and **compile-time safety**.

---

# 1. Find Methods (Avoid)

**Find objects in scene. Slow — use sparingly, cache results.**

```csharp
// By type (searches entire scene)
AudioManager audio = FindObjectOfType<AudioManager>();

// By name (searches entire scene)
GameObject obj = GameObject.Find("AudioManager");

// By tag
GameObject player = GameObject.FindWithTag("Player");
```

> [!warning] Avoid in Update
> These are slow operations. Call once in Start/Awake and cache the result.

**Why avoid:**

- **Performance:** Expensive scene traversal, especially in `Update`
- **Fragility:** Hard to trace where references come from
- **Runtime errors:** Crashes if objects are renamed or moved
- **No compile-time safety:** You won't know it's broken until runtime

**When acceptable:** Initialization phase (Awake/Start), truly dynamic lookups where you don't know what exists.

---

# 2. Direct Reference / SerializeField (Foundation)

**Explicit inspector assignment. Use for known, stable dependencies.**

```csharp
public class Player : MonoBehaviour
{
    [SerializeField] private AudioManager _audio;  // Assigned in Inspector
    [SerializeField] private InputManager _input;

    public void Jump()
    {
        _audio.PlaySound("Jump");
    }
}
```

**When to use:**

- Components on same GameObject
- Parent-child relationships
- Stable, known dependencies

**Pros:**

- Simple, fast, type-safe
- Visible in Inspector (clear dependencies)
- Debuggable (see what's assigned)

**Cons:**

- Tight coupling (Player knows AudioManager exists)
- Breaks if reference missing (null reference exception)
- Passing references through deep hierarchies is tedious

**The "Single Entry Point" pattern** (from previous PracticAPI video) mitigates this by centralizing all references in one place, creating a "manifest" of dependencies. However, manual wiring still becomes tedious at scale.

---

# 3. Singleton (The Trap)

**Global static instance. Convenient early, painful later.**

```csharp
public class AudioManager : MonoBehaviour
{
    public static AudioManager Instance { get; private set; }

    private void Awake()
    {
        if (Instance == null)
        {
            Instance = this;
            DontDestroyOnLoad(gameObject);
        }
        else
        {
            Destroy(gameObject);
        }
    }

    public void PlaySound(string name) { /* ... */ }
}

// Usage anywhere
AudioManager.Instance.PlaySound("Explosion");
```

**Why it feels good:**

- Accessible anywhere
- No wiring needed
- Fast to prototype

**Why it becomes a problem:**

### Memory Leaks
Static references never release. Menu data stays loaded during gameplay because the static reference keeps it alive.

### Hidden Dependencies
You don't know a class needs AudioManager until it runs and crashes. No compile-time visibility of what depends on what.

### Tight Coupling
Any class can access any singleton. You cannot restrict "only gameplay code can access AudioManager."

### Testing Nightmare
Can't substitute mock implementations. All tests run against the real singleton.

> [!warning] The Verdict
> **Fine for game jams and prototypes. Dangerous at scale.**

---

# 4. Service Locator (Professional)

**A central registry mapping types to instances. Manual dependency injection.**

```csharp
public static class Services
{
    private static Dictionary<Type, object> _services = new();

    public static void Register<T>(T service) => _services[typeof(T)] = service;
    public static void Unregister<T>() => _services.Remove(typeof(T));
    public static T Get<T>() => (T)_services[typeof(T)];

    public static bool TryGet<T>(out T service)
    {
        if (_services.TryGetValue(typeof(T), out var obj))
        {
            service = (T)obj;
            return true;
        }
        service = default;
        return false;
    }
}
```

**Registration (in initialization):**

```csharp
public class GameBootstrap : MonoBehaviour
{
    [SerializeField] private AudioManager _audio;
    [SerializeField] private InputManager _input;

    private void Awake()
    {
        Services.Register<IAudioService>(_audio);
        Services.Register<IInputService>(_input);
    }

    private void OnDestroy()
    {
        Services.Unregister<IAudioService>();
        Services.Unregister<IInputService>();
    }
}
```

**Usage:**

```csharp
public class Player : MonoBehaviour
{
    private IAudioService _audio;

    private void Awake()
    {
        _audio = Services.Get<IAudioService>();
    }

    public void Jump()
    {
        _audio.PlaySound("Jump");
    }
}
```

## Advantages Over Singleton

- **Memory management:** Services can be unregistered (memory freed)
- **Access control:** Register interface → implementation (hide internals)
- **Explicit registration:** You see what's available during initialization
- **Testable:** Swap in mock implementations

## Disadvantage

Still "pulls" dependencies — classes ask for what they need. No compile-time guarantee the service exists.

See [[Layered Architecture in Unity]] for full implementation.

---

# 5. Dependency Injection (Industry Standard)

**Automatic dependency resolution using reflection or code generation.**

## The Key Insight

Instead of **asking** for dependencies (pull), classes **declare** what they need (push). The DI container provides them automatically.

```csharp
// Service Locator: class asks for dependencies
public class EnemySpawner : MonoBehaviour
{
    private IObjectPool _pool;

    private void Awake()
    {
        _pool = Services.Get<IObjectPool>();  // Pulls dependency
    }
}

// DI: class receives dependencies
public class EnemySpawner : MonoBehaviour
{
    private IObjectPool _pool;

    [Inject]  // DI container provides this automatically
    public void Construct(IObjectPool pool)
    {
        _pool = pool;
    }
}
```

**Benefits:**

- Class never searches for anything — dependencies arrive automatically
- Compile-time errors if dependency not registered (container fails to resolve)
- Clear declaration of what a class needs
- Easy to test (inject mocks)

## DI Frameworks

| Framework      | Focus       | Notes                                      |
| -------------- | ----------- | ------------------------------------------ |
| **VContainer** | Performance | Faster, lighter, recommended for most      |
| **Zenject**    | Features    | More powerful, steeper learning curve      |

See [[Essential Unity Packages and Assets]] for VContainer setup.

## Lifetime Management

DI containers handle **creation** and **lifetime** differently for different types:

| Type                         | Who Creates          | Who Manages Lifetime          | Registration                            |
| ---------------------------- | -------------------- | ----------------------------- | --------------------------------------- |
| **Plain C# class**           | Container            | Container (Singleton/Transient)| `Register<T>(Lifetime.Singleton)`       |
| **MonoBehaviour in scene**   | You (scene/prefab)   | Unity (scene lifetime)        | `RegisterComponentInHierarchy<T>()`     |
| **MonoBehaviour prefab**     | Container instantiates| Container holds reference     | `RegisterComponentInNewPrefab<T>()`     |

**Key insight:** For MonoBehaviours in a Persistent scene, **you** manage lifetime (by not unloading the scene). The DI container just **finds** them and **injects** them.

```csharp
public class GameLifetimeScope : LifetimeScope
{
    protected override void Configure(IContainerBuilder builder)
    {
        // Plain C# — container CREATES and OWNS
        builder.Register<HealthSystem>(Lifetime.Singleton);
        builder.Register<InventorySystem>(Lifetime.Singleton);

        // MonoBehaviours in Persistent scene — container FINDS and INJECTS
        // (You created them in the scene; container just wires them up)
        builder.RegisterComponentInHierarchy<AudioManager>().As<IAudioService>();
        builder.RegisterComponentInHierarchy<InputManager>();
    }
}
```

### Constructor Parameters

For classes with custom constructor parameters:

```csharp
public class DamageCalculator
{
    public DamageCalculator(float baseDamage, HealthSystem health) { }
}

// Registration:
builder.Register<DamageCalculator>(Lifetime.Singleton)
    .WithParameter("baseDamage", 10f);  // Explicit value
// HealthSystem is auto-resolved because it's registered
```

---

# Advanced Architectural Safeguards

DI is not just about convenience — it's about **enforcing architecture**. Three layers of protection:

## Layer 1: Context Separation

**Concept:** Separate logic into "Scene Contexts" (e.g., Core vs. Gameplay).

**Rule:** **Gameplay** can see **Core**, but **Core** cannot see **Gameplay**.

**Result:** Prevents "upward coupling." A generic audio service (Core) can never accidentally depend on a specific enemy unit (Gameplay).

## Layer 2: Assembly Definitions (.asmdef)

**Concept:** Compile code into separate DLLs.

**Benefit:** Moves runtime errors to **compile-time errors**. If a script in the "Core" assembly tries to reference "Gameplay", the code won't compile. This physically prevents bad architectural decisions.

## Layer 3: Circular Dependency Prevention

**Concept:** The "bug that is a feature."

**Scenario:** Class A needs Class B, and Class B needs Class A.

**DI Behavior:** The DI system detects this infinite loop during initialization and throws an immediate error.

**Insight:** Forces you to refactor "spaghetti code" immediately, ensuring a clean, linear dependency tree (waterfall structure).

---

# Interfaces for Hiding Implementations

Interfaces allow you to expose only what's needed:

```csharp
public interface IAudioService
{
    void PlaySound(string name);
    void PlayMusic(string name);
}

public class AudioManager : MonoBehaviour, IAudioService
{
    // Public interface
    public void PlaySound(string name) { /* ... */ }
    public void PlayMusic(string name) { /* ... */ }

    // Hidden internals
    private void LoadAudioClip(string name) { /* ... */ }
    private void UnloadUnusedClips() { /* ... */ }
}
```

**Registration:**

```csharp
Services.Register<IAudioService>(_audio);  // Only IAudioService is visible
// or
builder.RegisterComponentInHierarchy<AudioManager>().As<IAudioService>();
```

**Benefits:**

- Receiver can't see private methods
- Can't accidentally call internal logic
- Easier to swap implementations (different audio backends)

---

# When to Use What

| Project Size                  | Recommendation              |
| ----------------------------- | --------------------------- |
| Game jam / prototype          | Singletons fine             |
| Small game (solo/small team)  | Service Locator             |
| Medium+ game (team, long-term)| Dependency Injection        |

## The Tradeoff

| Approach                 | Overhead                 | Benefit                              | When                                                 |
| ------------------------ | ------------------------ | ------------------------------------ | ---------------------------------------------------- |
| **Direct reference**     | None                     | Simple, debuggable                   | Components on same GameObject, stable relationships  |
| **Service Locator**      | Minimal (~50 lines)      | Explicit, you see registration       | Solo/small team, want full control                   |
| **Dependency Injection** | Package + learning curve | Automatic wiring, enforced contracts | Team projects, large codebase, long-term maintenance |

**DI adds complexity.** For a game jam or prototype, it's overkill. You're adding infrastructure for a problem you don't have yet.

**DI pays off when:**

- Multiple developers need to understand dependencies at a glance
- You want compile-time errors when dependencies are missing
- You need to swap implementations (testing, platform-specific)
- Codebase is large enough that manual wiring becomes error-prone

**Rule of thumb:** Start with Service Locator. Move to DI when you feel pain from manual registration or hidden dependencies.

---

# Key Takeaways

1. **Find methods** are slow and fragile — avoid except for initialization
2. **SerializeField** is the foundation — explicit and visible
3. **Singletons** work for prototypes but don't scale
4. **Service Locator** provides explicit control without a framework
5. **Dependency Injection** enforces architecture and catches errors at compile-time
6. **Interfaces** hide implementation details and enable swapping
7. **Start simple** — add complexity only when you feel the pain

---

# Related

- [[Communication Patterns in Unity]] — How to broadcast events and notifications
- [[Layered Architecture in Unity]] — Full Service Locator implementation
- [[Single Entry Point in Unity]] — Centralized initialization pattern
- [[Essential Unity Packages and Assets]] — VContainer setup
- [[Scene Management in Unity]] — Persistent scene managers and DI integration
