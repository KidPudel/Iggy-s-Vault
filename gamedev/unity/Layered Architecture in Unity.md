# Layered Architecture in Unity

> [!quote] The Core Insight Separate what IS true (simulation) from what SHOWS the truth (presentation).

---

# The Problem

Without architecture, code becomes a web where everything depends on everything. Change one thing, break ten others. Can't test in isolation. Can't reason about flow.

**The fix:** Layers with clear responsibilities and communication rules.

---

# The Three Layers

```
┌─────────────────────────────────────────────────────────────────┐
│                        SYSTEMS                                  │
│                    "What IS true?"                              │
├─────────────────────────────────────────────────────────────────┤
│  Pure data and logic. Zero visuals/UI/sound. Isolated.          │
│  Plain C# classes. Raises events. Testable without Unity.       │
│                                                                 │
│  Example: HealthSystem knows health values and damage formulas. │
│  Knows NOTHING about death animations or loot drops.            │
└─────────────────────────────────────────────────────────────────┘
                              │
                              │ Managers CALL systems
                              │ Systems RAISE events
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                        MANAGERS                                 │
│                "What happens when X occurs?"                    │
├─────────────────────────────────────────────────────────────────┤
│  Game-specific orchestration. Coordinates ripple effects.       │
│  MonoBehaviour or plain C#. Connects systems to presentation.   │
│                                                                 │
│  Example: CombatManager hears "Died" event →                    │
│  removes from turn order, rolls loot, notifies quests.          │
└─────────────────────────────────────────────────────────────────┘
                              │
                              │ Managers TELL components
                              │ Components CALL managers/systems
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                       COMPONENTS                                │
│                "I know what X means for MY type"                │
├─────────────────────────────────────────────────────────────────┤
│  Type-specific behavior. Owns visuals. Handles itself.          │
│  MonoBehaviour. Manager doesn't need to know internals.         │
│                                                                 │
│  Example: Door knows what "interact" means for a door.          │
│  Manager just says "Interact()", door figures out the rest.     │
└─────────────────────────────────────────────────────────────────┘
```

|Layer|Analogy|Responsibility|
|---|---|---|
|**System**|The Script|What IS true|
|**Manager**|The Director|WHEN things happen, WHO does WHAT|
|**Component**|The Actor|HOW to perform its part|

---

# Systems

Pure logic. No Unity dependencies. Plain C# classes.

```csharp
// Pure C# class — no MonoBehaviour
public class HealthSystem
{
    private Dictionary<int, int> _health = new();
    
    public event Action<int, int, int> OnHealthChanged;  // id, old, new
    public event Action<int> OnDied;
    
    public void Register(int entityId, int maxHealth)
    {
        _health[entityId] = maxHealth;
    }
    
    public void Unregister(int entityId)
    {
        _health.Remove(entityId);
    }
    
    public void Damage(int entityId, int amount)
    {
        if (!_health.TryGetValue(entityId, out int current)) return;
        
        int newHealth = Mathf.Max(0, current - amount);
        _health[entityId] = newHealth;
        
        OnHealthChanged?.Invoke(entityId, current, newHealth);
        
        if (newHealth == 0)
            OnDied?.Invoke(entityId);
    }
    
    public int Get(int entityId) => _health.GetValueOrDefault(entityId);
}
```

**Systems are truth machines:**

- Hold data
- Apply rules
- Announce changes via events
- **That's it**

## Why Plain C#?

- Testable without Unity (NUnit, no Play Mode)
- No lifecycle confusion
- Clear dependencies
- Portable to other contexts

---

# Managers

Orchestrate responses to events. May need Unity capabilities.

```csharp
public class CombatManager : MonoBehaviour
{
    private void OnEnable()
    {
        GameSystems.Health.OnDied += HandleDeath;
    }
    
    private void OnDisable()
    {
        GameSystems.Health.OnDied -= HandleDeath;
    }
    
    private void HandleDeath(int entityId)
    {
        // Orchestrate IN ORDER
        RemoveFromTurnOrder(entityId);
        SpawnLoot(entityId);
        NotifyQuests(entityId);
        CheckCombatEnd();
    }
}
```

**When you need a Manager:**

- Ripple effects across systems
- Order matters
- Need Unity features (coroutines, spawning, timers)
- Conditional logic ("if boss, trigger cutscene")

**Manager capabilities Systems lack:**

|Capability|System|Manager|
|---|---|---|
|Coroutines|❌|✅|
|Scene/hierarchy access|❌|✅|
|Update / FixedUpdate|❌|✅|
|Instantiate prefabs|❌|✅|

---

# Components

MonoBehaviours on GameObjects. Handle type-specific behavior.

```csharp
public class Enemy : MonoBehaviour
{
    [SerializeField] private int _maxHealth = 100;
    
    private int _entityId;
    
    private void Awake()
    {
        _entityId = GetInstanceID();
    }
    
    private void OnEnable()
    {
        GameSystems.Health.Register(_entityId, _maxHealth);
        GameSystems.Health.OnDied += HandleDeath;
    }
    
    private void OnDisable()
    {
        GameSystems.Health.Unregister(_entityId);
        GameSystems.Health.OnDied -= HandleDeath;
    }
    
    private void HandleDeath(int entityId)
    {
        if (entityId != _entityId) return;
        
        // Visual death — this component knows HOW to die
        PlayDeathAnimation();
        DisableCollider();
    }
}
```

---

# Source of Truth

**"Truth"** = where authoritative state lives.

|Owner|When|Example|
|---|---|---|
|**System**|Shared pool of data|`HealthSystem` owns all entity HP|
|**Component**|Self-contained, local state|`Door` owns its open/closed state|
|**Manager**|Rare — game-wide state only it tracks|`DayCycleManager` owns current time|

## Two Kinds of Components

|Kind|Truth lives...|Component's job|
|---|---|---|
|**Component-as-Truth**|In itself|Own state, BE the feature|
|**Component-as-View**|In a System|Display state, request changes|

Most simple objects (doors, triggers, decorations) are **Component-as-Truth**. They don't need Systems.

**Component-as-View** exists when a System owns the data:

```
HealthSystem owns entity health
    ↓
HealthBar (Component-as-View) displays it
```

---

# Communication Rules

```
┌─────────────────────┬───────────────────────────────────────────┐
│  FROM → TO          │ METHOD                                    │
├─────────────────────┼───────────────────────────────────────────┤
│  Component → System │ Direct call                               │
│  Component → Manager│ Direct call or event                      │
│  Manager → System   │ Direct call                               │
│  Manager → Component│ Direct call or event                      │
│  System → anything  │ EVENT ONLY                                │
│  System → System    │ NEVER                                     │
└─────────────────────┴───────────────────────────────────────────┘
```

**Why Systems only raise events:** They're pure truth — announce changes, don't control flow.

**Why Systems never call Systems:** Hidden dependencies. If HealthSystem needs to affect InventorySystem, a Manager coordinates.

---

# Accessing Systems

How do Components and Managers get references to Systems?

## Option 1: Static Container

> Or singleton ([[Common Patterns Cookbook in Unity]])

Simplest approach. Systems are created once at startup.

```csharp
public static class GameSystems
{
    public static HealthSystem Health { get; } = new();
    public static InventorySystem Inventory { get; } = new();
    public static QuestSystem Quests { get; } = new();
}

// Usage
GameSystems.Health.Damage(entityId, 10);
```

**Pros:** Simple, fast, IDE autocomplete works. **Cons:** Can't swap implementations, systems live forever.

## Option 2: Service Locator

More flexible. Systems register/unregister dynamically.

```csharp
public static class Services
{
    private static readonly Dictionary<Type, object> _services = new();
    
    public static void Register<T>(T service)
    {
        _services[typeof(T)] = service;
    }
    
    public static void Register<TInterface, TImplementation>(TImplementation service) 
        where TImplementation : TInterface
    {
        _services[typeof(TInterface)] = service;
    }
    
    public static void Unregister<T>()
    {
        _services.Remove(typeof(T));
    }
    
    public static T Get<T>()
    {
        if (_services.TryGetValue(typeof(T), out var service))
            return (T)service;
        throw new InvalidOperationException($"Service {typeof(T)} not registered");
    }
    
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
    
    public static void Clear() => _services.Clear();
}
```

**Interface-based registration** hides implementation details:

```csharp
// Registration (in bootstrap)
Services.Register<IAudioService, AudioService>(new AudioService());

// Usage (consumer only sees interface)
Services.Get<IAudioService>().PlaySound(clip);
// Consumer can't access AudioService internals — only IAudioService methods
```

**Lifecycle management:**

```csharp
// In GameInitiator or Bootstrap
private void InitializeSystems()
{
    Services.Register(new HealthSystem());
    Services.Register(new InventorySystem());
    Services.Register<IAudioService, AudioService>(new AudioService());
}

private void OnDestroy()
{
    Services.Clear();  // Clean up on game end
}
```

**Pros:** Swap implementations (testing), memory managed, interface hiding. **Cons:** Runtime errors if service missing, no compile-time checking.

## Option 3: Dependency Injection

For larger projects, use a DI framework (VContainer, Zenject). See [[Communication Patterns in Unity]] for details.

## Which to Choose

|Project|Recommendation|
|---|---|
|Prototype|Static Container|
|Small game|Static Container or Service Locator|
|Team project|Service Locator or DI|
|Need testing/mocking|Service Locator (interface-based) or DI|

---

# When to Create What

```
Q1: Is state in a shared pool (multiple entities)?
    YES → System owns the pool
    NO  → Continue...

Q2: Does this event ripple across multiple unrelated things?
    YES → Manager coordinates
    NO  → Continue...

Q3: Does logic need Unity capabilities?
    YES → Manager or Component
    NO  → System or static helper

Q4: Is this just local state for one object?
    YES → Component-as-Truth (owns itself)
```

## Examples

|Feature|Shared Pool?|Ripple?|Result|
|---|---|---|---|
|All-entity health|Yes|—|HealthSystem|
|Per-enemy health|No|—|Component-as-Truth|
|Inventory|Yes|Maybe|System + Component-as-View|
|Door interaction|No|No|Component-as-Truth|
|Enemy death|—|Yes|Manager coordinates|
|Day/night cycle|No|Yes|Manager (needs Update)|

---

# Keep It Simple

> [!warning] Architecture is a tool, not a religion.

**Signs of over-architecture:**

|Smell|Reality|
|---|---|
|"System because multiple things call it"|Is state pooled? If no, just a helper.|
|"Manager" that forwards one call|Call directly.|
|"Event bus" to read player position|Just read it.|
|System for everything|Most things are Component-as-Truth.|

Most game objects are simple. A door, trigger, decoration — just own themselves. Architecture exists for genuinely complex cases.

---

# The 70/30 Rule

Jonas Tyroller's guideline:

- **~70%** isolated, reusable systems/components
- **~30%** glue code (managers, game-specific wiring)

Don't try to abstract everything. Generalize where it makes sense. Game-specific glue is fine.

---

# Project Structure

```
Assets/
├── Scripts/
│   ├── Systems/              # Plain C# classes
│   │   ├── GameSystems.cs
│   │   ├── HealthSystem.cs
│   │   └── InventorySystem.cs
│   ├── Managers/             # MonoBehaviours
│   │   ├── CombatManager.cs
│   │   └── DayCycleManager.cs
│   ├── Components/           # MonoBehaviours on GameObjects
│   │   ├── Health.cs
│   │   ├── Movement.cs
│   │   └── Door.cs
│   └── Helpers/              # Static utilities
│       └── DamageCalculator.cs
```

---

# Checklist

**Systems:**

- [ ] Plain C# class (no MonoBehaviour)
- [ ] Zero Unity dependencies (except maybe Mathf)
- [ ] Zero presentation (no GameObjects, UI, audio)
- [ ] Events only for output
- [ ] Exists because: shared data pool

**Managers:**

- [ ] Exists because: ripple effects OR needs Unity features
- [ ] Subscribes to system events
- [ ] Unsubscribes in OnDisable
- [ ] Public methods describe what happens, not how

**Components:**

- [ ] Component-as-Truth: owns state, IS the feature
- [ ] Component-as-View: displays, requests mutations
- [ ] Handles type-specific behavior (interfaces help)

**Sanity:**

- [ ] Could this component just own itself?
- [ ] Is there actually a shared pool?
- [ ] Could events alone handle this?

---

# Key Takeaways

1. **Systems** = pure truth (plain C#, events only)
2. **Managers** = orchestration (when order/ripples matter)
3. **Components** = actors (handle their own behavior)
4. **Most things are simple** — Component-as-Truth
5. **70/30 rule** — reusable systems, some glue
6. **Don't over-architect** — add layers when complexity demands

> [!quote] The Test Can you explain your game's flow by reading just the manager's public methods?