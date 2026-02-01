# Communication Patterns in Unity

> [!quote] The Core Insight
> Unity has no built-in signals. Choose your communication pattern based on coupling needs and cardinality (one-to-many).

---

# The Problem

Objects need to broadcast notifications: "something happened, who cares?" This is **one-to-many communication** — health changes, player dies, button clicked. Multiple listeners may respond, or none.

**This document covers event patterns.** For service access (getting references to managers), see [[Reference Objects in Unity]].

---

# Direct Reference

**Tightest coupling. Use when you know exactly who should respond.**

```csharp
public class Player : MonoBehaviour
{
    [SerializeField] private Health _health;  // Assigned in Inspector

    private void Start()
    {
        // Subscribe to events on known dependency
        _health.OnDied += HandleDeath;
    }

    private void OnDestroy()
    {
        _health.OnDied -= HandleDeath;
    }

    private void HandleDeath() { /* ... */ }
}
```

**When to use:**

- Components on same GameObject
- Parent-child relationships
- Stable, known relationships where the subscriber knows the publisher

**Pros:** Simple, fast, type-safe, debuggable  
**Cons:** Tight coupling, requires knowing the event source

**Note:** Direct references are also used for dependency access. For that use case, see [[Reference Objects in Unity]].

---

# GetComponent

**Find event sources at runtime. Cache the result.**

```csharp
public class Enemy : MonoBehaviour
{
    private Health _health;

    private void Awake()
    {
        _health = GetComponent<Health>();
        _health.OnDied += HandleDeath;
    }

    private void OnDestroy()
    {
        _health.OnDied -= HandleDeath;
    }

    private void HandleDeath() { /* ... */ }
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

// Safe check
if (TryGetComponent<Health>(out var health))
{
    health.OnDied += HandleDeath;
}
```

**When to use:** Finding event sources on related GameObjects (same, parent, children).

> [!warning] Cache Results
> GetComponent searches the GameObject's component list. Call once in Awake and cache the result.

**Note:** GetComponent is also used for dependency access. For that use case, see [[Reference Objects in Unity]].

---

# C# Events

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

**Why this pattern:**

- Multiple listeners can subscribe (UI, audio, effects)
- Publisher doesn't know who's listening
- Type-safe, compile-time checked

> [!danger] Always Unsubscribe
> Subscribe in `OnEnable`, unsubscribe in `OnDisable`. Forgetting causes memory leaks and null reference errors.

**Event signatures:**

```csharp
public event Action OnSimpleEvent;              // No parameters
public event Action<int> OnValueChanged;        // One parameter
public event Action<int, float> OnDamage;       // Multiple parameters
public event EventHandler OnStandardEvent;      // .NET standard (sender, args)
```

---

# UnityEvent

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

Designers wire up responses in Inspector — no code needed for subscribers.

**When to use:**

- UI interactions
- Designer-configurable responses
- When non-programmers need to set up behavior
- Rapid prototyping

**Cons:** Slower than C# events, harder to debug, serialization limitations

---

# ScriptableObject Event Channels

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

**Usage:**

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
- Easily testable (raise event manually in editor)
- Survives scene loads

**When to use:**

- Cross-scene communication
- Core/Persistent systems broadcasting to gameplay scenes
- Decoupling major systems (score, achievements, analytics)

See [[ScriptableObjects]] for more patterns.

---

# Static Events / Event Bus

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

**Pros:** Simple, accessible anywhere  
**Cons:** Global state, hard to track listeners, testing difficulties, memory never released

**When to use:**

- Truly global events (game pause, scene transitions)
- When ScriptableObject events feel like overkill
- Prototypes and game jams

**Prefer ScriptableObject events** for anything that might need to be scoped or cleared between scenes.

---

# Interfaces

**Define contracts for polymorphic event handlers.**

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

**Polymorphic responses:**

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

**Event-driven caller:**

```csharp
public class Weapon : MonoBehaviour
{
    public void Hit(Collider other)
    {
        // Doesn't care about concrete type
        if (other.TryGetComponent<IDamageable>(out var target))
        {
            target.TakeDamage(_damage);
        }
    }
}
```

**Why interfaces for events:**

- Weapon doesn't need to know the concrete type
- New damageable types work automatically (barrels, props, walls)
- Clean separation between caller and responders

**Note:** Interfaces are also used for dependency access (hiding implementations). For that use case, see [[Reference Objects in Unity]].

---

# Choosing a Communication Pattern

| Relationship                      | Pattern                      |
| --------------------------------- | ---------------------------- |
| Same GameObject                   | Direct reference + C# Events |
| Parent-child                      | GetComponent + C# Events     |
| Known dependency                  | [SerializeField] + C# Events |
| One-to-many, same scene           | C# Events                    |
| Designer-configurable             | UnityEvent                   |
| Cross-scene / decoupled systems   | ScriptableObject Events      |
| Polymorphic behavior              | Interfaces                   |
| Truly global (rare)               | Static Events                |

---

# Godot Comparison

| Godot               | Unity                  |
| ------------------- | ---------------------- |
| `signal`            | C# `event Action`      |
| `emit_signal()`     | `OnEvent?.Invoke()`    |
| `connect()`         | `+= handler`           |
| `disconnect()`      | `-= handler`           |
| Built-in to Object  | Must declare yourself  |
| Groups              | Tags or static events  |

**Key difference:** Godot has signals built into every Object. Unity requires explicit event declaration or alternative patterns.

---

# Key Takeaways

1. **Direct references** for stable, known event sources
2. **C# events** for one-to-many within same context
3. **ScriptableObject events** for cross-scene/decoupled communication
4. **UnityEvent** for designer-configurable responses
5. **Always unsubscribe** from events in OnDisable
6. **Cache** GetComponent results — don't search repeatedly
7. **Interfaces** enable polymorphic responses without tight coupling
8. **Static events** are global state — use sparingly

---

# Related

- [[Reference Objects in Unity]] — How to access services (AudioManager, InputManager)
- [[ScriptableObjects]] — Shared data and event channels
- [[Scene Management in Unity]] — Cross-scene communication patterns
- [[Single Entry Point in Unity]] — Centralized initialization
