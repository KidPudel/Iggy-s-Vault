# Communication Patterns

> [!quote] The Core Insight Unity has no built-in signals. Choose your communication pattern based on coupling needs.

---

## The Problem

Objects need to talk to each other. Options range from tight coupling (direct references) to loose coupling (events). No single answer â€” pick based on relationship.

---

## Direct Reference

**Tightest coupling. Use for known, stable relationships.**

```csharp
public class Player : MonoBehaviour
{
    [SerializeField] private Health _health;  // Assigned in Inspector
    [SerializeField] private Weapon _weapon;
    
    public void Attack()
    {
        _weapon.Fire();
    }
}
```

**When to use:**

- Components on same GameObject
- Parent-child relationships
- Stable, known dependencies

**Pros:** Simple, fast, type-safe, debuggable **Cons:** Tight coupling, breaks if reference missing

---

## GetComponent

**Find components at runtime. Cache the result.**

```csharp
public class Enemy : MonoBehaviour
{
    private Health _health;
    private Rigidbody _rb;
    
    private void Awake()
    {
        _health = GetComponent<Health>();
        _rb = GetComponent<Rigidbody>();
    }
}
```

**Variants:**

```csharp
// On same GameObject
GetComponent<T>()

// On children (includes self)
GetComponentInChildren<T>()

// On parent (includes self)
GetComponentInParent<T>()

// Multiple components
GetComponents<T>()
GetComponentsInChildren<T>()

// Safe check
if (TryGetComponent<Health>(out var health))
{
    health.TakeDamage(10);
}
```

---

## Find Methods

**Find objects in scene. Slow â€” use sparingly, cache results.**

```csharp
// By type (searches entire scene)
Player player = FindObjectOfType<Player>();

// By name (searches entire scene)
GameObject obj = GameObject.Find("Player");

// By tag
GameObject obj = GameObject.FindWithTag("Player");
GameObject[] enemies = GameObject.FindGameObjectsWithTag("Enemy");
```

> [!warning] Avoid in Update These are slow operations. Call once in Start/Awake and cache the result.

---

## C# Events

**Decoupled one-to-many communication. The core pattern.**

```csharp
// Publisher
public class Health : MonoBehaviour
{
    public event Action<int> OnHealthChanged;
    public event Action OnDied;
    
    private int _current;
    
    public void TakeDamage(int amount)
    {
        _current -= amount;
        OnHealthChanged?.Invoke(_current);
        
        if (_current <= 0)
            OnDied?.Invoke();
    }
}

// Subscriber
public class HealthUI : MonoBehaviour
{
    [SerializeField] private Health _health;
    
    private void OnEnable()
    {
        _health.OnHealthChanged += UpdateDisplay;
        _health.OnDied += ShowDeathScreen;
    }
    
    private void OnDisable()
    {
        _health.OnHealthChanged -= UpdateDisplay;
        _health.OnDied -= ShowDeathScreen;
    }
    
    private void UpdateDisplay(int health) { /* ... */ }
    private void ShowDeathScreen() { /* ... */ }
}
```

> [!danger] Always Unsubscribe Subscribe in `OnEnable`, unsubscribe in `OnDisable`. Forgetting causes memory leaks and null reference errors.

---

## UnityEvent

**Inspector-configurable events. Designer-friendly.**

```csharp
using UnityEngine.Events;

public class Button : MonoBehaviour
{
    [SerializeField] private UnityEvent _onClick;
    [SerializeField] private UnityEvent<int> _onValueChanged;
    
    public void Click()
    {
        _onClick?.Invoke();
    }
}
```

Designers wire up responses in Inspector, no code needed for subscribers.

**When to use:**

- UI interactions
- Designer-configurable responses
- When non-programmers need to set up behavior

**Cons:** Slower than C# events, harder to debug, serialization limitations

---

## ScriptableObject Event Channels

**Scene-independent events. Best for cross-system communication.**

```csharp
// Event definition
[CreateAssetMenu(menuName = "Events/Void Event")]
public class VoidEvent : ScriptableObject
{
    private Action _listeners;
    
    public void Register(Action listener) => _listeners += listener;
    public void Unregister(Action listener) => _listeners -= listener;
    public void Raise() => _listeners?.Invoke();
}

// With data
[CreateAssetMenu(menuName = "Events/Int Event")]
public class IntEvent : ScriptableObject
{
    private Action<int> _listeners;
    
    public void Register(Action<int> listener) => _listeners += listener;
    public void Unregister(Action<int> listener) => _listeners -= listener;
    public void Raise(int value) => _listeners?.Invoke(value);
}
```

```csharp
// Publisher
public class Player : MonoBehaviour
{
    [SerializeField] private IntEvent _onScoreChanged;
    
    public void AddScore(int points)
    {
        _score += points;
        _onScoreChanged.Raise(_score);
    }
}

// Subscriber
public class ScoreUI : MonoBehaviour
{
    [SerializeField] private IntEvent _onScoreChanged;
    
    private void OnEnable() => _onScoreChanged.Register(UpdateDisplay);
    private void OnDisable() => _onScoreChanged.Unregister(UpdateDisplay);
    
    private void UpdateDisplay(int score) => _text.text = score.ToString();
}
```

**Why this pattern:**

- No direct reference between Player and ScoreUI
- Works across scenes (both reference same SO asset)
- Easily testable (raise event manually)

See [[ScriptableObjects]] for more patterns.

---

## Interfaces

**Define contracts. Enable polymorphism without inheritance.**

```csharp
public interface IDamageable
{
    void TakeDamage(int amount);
}

public interface IInteractable
{
    void Interact();
    string GetPrompt();
}
```

```csharp
public class Enemy : MonoBehaviour, IDamageable
{
    public void TakeDamage(int amount) { /* ... */ }
}

public class Barrel : MonoBehaviour, IDamageable
{
    public void TakeDamage(int amount) { /* ... */ }
}
```

```csharp
// Caller doesn't care about concrete type
public class Weapon : MonoBehaviour
{
    public void Hit(Collider other)
    {
        if (other.TryGetComponent<IDamageable>(out var target))
        {
            target.TakeDamage(_damage);
        }
    }
}
```

---

## Static Events / Event Bus

**Global events. Use sparingly.**

```csharp
public static class GameEvents
{
    public static event Action OnGamePaused;
    public static event Action OnGameResumed;
    public static event Action<int> OnScoreChanged;
    
    public static void Pause() => OnGamePaused?.Invoke();
    public static void Resume() => OnGameResumed?.Invoke();
    public static void ChangeScore(int score) => OnScoreChanged?.Invoke(score);
}
```

```csharp
// Subscriber
private void OnEnable() => GameEvents.OnGamePaused += HandlePause;
private void OnDisable() => GameEvents.OnGamePaused -= HandlePause;
```

**Pros:** Simple, accessible anywhere **Cons:** Global state, hard to track listeners, test difficulties

---

## Dependency Resolution

Events handle "something happened, who cares?" (one-to-many). **Dependency resolution** handles "I need access to X" (one-to-one service access).

### The Evolution

|Stage|Technique|Verdict|
|---|---|---|
|1. Find|`GameObject.Find()`, `FindObjectOfType()`|Avoid — slow, fragile, runtime errors|
|2. Serialized|`[SerializeField]` in Inspector|Good foundation — explicit, visible|
|3. Singleton|`public static Instance`|Trap — works for prototypes, scales poorly|
|4. Service Locator|`Services.Get<T>()`|Professional — controlled access, memory managed|
|5. Dependency Injection|Constructor injection|Industry standard — automatic, enforced|

### Why Singletons Become Problems

```csharp
// Looks convenient
AudioManager.Instance.PlaySound(clip);
```

**The traps:**

- **Memory:** Static references never release (menu data stays during gameplay)
- **Hidden dependencies:** You don't know a class needs AudioManager until it crashes
- **No access control:** Any class can call any singleton
- **Testing:** Can't substitute mock implementations

Fine for game jams. Dangerous at scale.

### Service Locator (Manual DI)

A dictionary mapping types to instances. See [[Layered Architecture in Unity]] for full implementation.

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

**Advantages over Singleton:**

- Services can be unregistered (memory freed)
- Register interface → implementation (hide internals)
- Explicit registration during initialization

**Disadvantage:** Still "pulls" dependencies (code asks for what it needs).

### Dependency Injection (Automatic)

Instead of _asking_ for dependencies, classes _declare_ what they need:

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

**Key insight:** The class never searches for anything. Dependencies arrive automatically.

### DI Frameworks

|Framework|Focus|Notes|
|---|---|---|
|**VContainer**|Performance|Faster, lighter, recommended for most projects|
|**Zenject**|Features|More powerful, steeper learning curve|

### When to Use What

|Project Size|Recommendation|
|---|---|
|Game jam / prototype|Singletons fine|
|Small game (solo/small team)|Service Locator|
|Medium+ game (team, long-term)|Dependency Injection|

See [[Essential Unity Packages and Assets]] for VContainer setup.

---

## Choosing a Pattern

### Communication (Events)

|Relationship|Pattern|
|---|---|
|Same GameObject|Direct reference / GetComponent|
|Parent-child|[SerializeField] or GetComponentInParent/Children|
|Known dependency|[SerializeField] direct reference|
|One-to-many, same scene|C# events|
|Designer-configurable|UnityEvent|
|Cross-scene / decoupled systems|ScriptableObject events|
|Polymorphic behavior|Interfaces|
|Truly global (rare)|Static events|

### Service Access (Dependencies)

|Situation|Pattern|
|---|---|
|Prototype / game jam|Singleton|
|Small project, explicit control|Service Locator|
|Team project, long-term|Dependency Injection|
|Need to swap implementations|Interface + Service Locator or DI|

---

## Godot Comparison

|Godot|Unity|
|---|---|
|`signal`|C# `event Action`|
|`emit_signal()`|`OnEvent?.Invoke()`|
|`connect()`|`+= handler`|
|`disconnect()`|`-= handler`|
|Built-in to every Object|Must declare yourself|
|Groups|Tags or static event bus|

**Key difference:** Godot has signals built into every Object. Unity requires explicit event declaration or alternative patterns.

---

## Key Takeaways

1. **Direct references** for stable, known relationships
2. **C# events** for one-to-many within same context
3. **ScriptableObject events** for decoupled cross-system communication
4. **Interfaces** for polymorphic contracts
5. **Always unsubscribe** from events in OnDisable
6. **Cache** GetComponent and Find results
7. **Singletons are a trap** — convenient early, painful later
8. **Service Locator** for explicit service access without DI framework
9. **Dependency Injection** for team/long-term projects