# Component-Based Design

> [!quote] The Core Insight Build entities by assembling components, not by inheriting from base classes.

---

## The Principle

**Composition over inheritance.**

Instead of:

```
Entity
├── Character
│   ├── Player
│   └── Enemy
│       ├── Goblin
│       └── Orc
└── Prop
    ├── Barrel
    └── Crate
```

Do:

```
GameObject + Health + Movement               = Player
GameObject + Health + Movement + AI          = Enemy
GameObject + Health                          = Barrel
GameObject + Interactable                    = Crate
```

Each component handles **one responsibility**. Combine them to create varied entities.

---

## Why This Matters

### The Inheritance Problem

```csharp
// Inheritance nightmare
class FlyingSwimmingElectricEnemy : ???
{
    // Does it inherit from FlyingEnemy, SwimmingEnemy, or ElectricEnemy?
    // C# only allows single inheritance
}
```

### The Component Solution

```
// Just add what you need
GameObject "WeirdCreature"
├── Health
├── FlyMovement
├── SwimMovement
├── ElectricAttack
└── EnemyAI
```

Want a non-flying version? Remove `FlyMovement`. No code changes.

---

## Single Responsibility

Each component does **one thing**.

```csharp
// BAD: One component does everything
public class Enemy : MonoBehaviour
{
    public void TakeDamage() { }
    public void Move() { }
    public void Attack() { }
    public void PlaySound() { }
    public void UpdateUI() { }
}

// GOOD: Separate responsibilities
public class Health : MonoBehaviour { }
public class Movement : MonoBehaviour { }
public class Attack : MonoBehaviour { }
// Sound and UI react to events from these
```

---

## Reusable Components

Write components that work on anything.

```csharp
public class Health : MonoBehaviour
{
    [SerializeField] private int _max = 100;
    
    public int Current { get; private set; }
    public int Max => _max;
    
    public event Action<int> OnChanged;
    public event Action OnDepleted;
    
    private void Awake() => Current = _max;
    
    public void TakeDamage(int amount)
    {
        Current = Mathf.Max(0, Current - amount);
        OnChanged?.Invoke(Current);
        
        if (Current == 0)
            OnDepleted?.Invoke();
    }
    
    public void Heal(int amount)
    {
        Current = Mathf.Min(_max, Current + amount);
        OnChanged?.Invoke(Current);
    }
}
```

Now `Health` works on Player, Enemy, Barrel, Boss, NPC — anything.

---

## Component Communication

Components on the same GameObject talk via:

### Direct Reference (GetComponent)

```csharp
public class DeathHandler : MonoBehaviour
{
    private Health _health;
    
    private void Awake()
    {
        _health = GetComponent<Health>();
    }
    
    private void OnEnable()
    {
        _health.OnDepleted += HandleDeath;
    }
    
    private void OnDisable()
    {
        _health.OnDepleted -= HandleDeath;
    }
    
    private void HandleDeath()
    {
        // Play animation, drop loot, destroy
        Destroy(gameObject, 2f);
    }
}
```

### RequireComponent

Enforce dependencies:

```csharp
[RequireComponent(typeof(Health))]
public class DeathHandler : MonoBehaviour
{
    // Health is guaranteed to exist
}
```

---

## Interfaces for Polymorphism

Components don't share a base class? Use interfaces.

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
public class Health : MonoBehaviour, IDamageable
{
    public void TakeDamage(int amount) { /* ... */ }
}

public class Shield : MonoBehaviour, IDamageable
{
    public void TakeDamage(int amount) { /* absorb damage */ }
}
```

```csharp
// Caller doesn't care which implementation
public class Weapon : MonoBehaviour
{
    private void OnHit(Collider other)
    {
        if (other.TryGetComponent<IDamageable>(out var target))
        {
            target.TakeDamage(_damage);
        }
    }
}
```

---

## Composition Patterns

### Behavior Assembly

```csharp
// Base entity just has common components
GameObject "BaseEnemy"
├── Health
├── Collider
└── Rigidbody

// Variants add specific behaviors
GameObject "MeleeEnemy"
├── (BaseEnemy components)
├── MeleeAttack
└── ChaseAI

GameObject "RangedEnemy"
├── (BaseEnemy components)
├── RangedAttack
└── KeepDistanceAI
```

Use Prefab Variants to create these efficiently. See [[Prefabs]].

### Dynamic Composition

Add/remove capabilities at runtime:

```csharp
// Player picks up jetpack
gameObject.AddComponent<FlyMovement>();

// Curse wears off
Destroy(GetComponent<Cursed>());

// Check capability
if (TryGetComponent<FlyMovement>(out var fly))
{
    fly.Ascend();
}
```

---

## Data vs Behavior Components

Not all components have behavior. Some just hold data:

```csharp
// Data component
public class Stats : MonoBehaviour
{
    [field: SerializeField] public int Strength { get; private set; }
    [field: SerializeField] public int Agility { get; private set; }
    [field: SerializeField] public int Intelligence { get; private set; }
}

// Behavior component reads it
public class MeleeDamageCalculator : MonoBehaviour
{
    private Stats _stats;
    
    private void Awake() => _stats = GetComponent<Stats>();
    
    public int Calculate() => _baseAttack + _stats.Strength * 2;
}
```

Or use ScriptableObjects for shared data. See [[ScriptableObjects]].

---

## The 70/30 Rule

Jonas Tyroller's guideline:

- **~70%** reusable, isolated components
- **~30%** glue code specific to this entity

```csharp
// 70% — Reusable
Health.cs
Movement.cs
Interactable.cs
Inventory.cs

// 30% — Glue for this specific entity
PlayerController.cs  // Wires Player-specific components together
BossAI.cs            // Boss-specific behavior using Health, Attack, etc.
```

Don't over-generalize. If something is only used once, it doesn't need to be a reusable system.

---

## Anti-Patterns

### God Component

```csharp
// BAD: One component does everything
public class Player : MonoBehaviour
{
    // 2000 lines of movement, combat, inventory, UI, audio...
}
```

Split by responsibility.

### Excessive Granularity

```csharp
// BAD: Too many tiny components
GameObject "Player"
├── MoveLeft
├── MoveRight
├── MoveForward
├── MoveBackward
├── Jump
├── DoubleJump
├── WallJump
└── ... (50 more)
```

Group related behavior. `Movement` can handle all movement.

### Inheritance in Components

```csharp
// BAD: Deep inheritance in components
public class MeleeAttack : Attack { }
public class SwordAttack : MeleeAttack { }
public class FireSwordAttack : SwordAttack { }
```

Prefer composition. Use ScriptableObjects for attack data variations.

---

## Godot Comparison

|Godot|Unity|
|---|---|
|Node inheritance + child composition|Component composition|
|`extends CharacterBody3D`|`RequireComponent` + composition|
|Multiple scripts per node (possible)|Multiple components per GameObject (normal)|
|Scenes as reusable compositions|Prefabs as reusable compositions|

Godot is a hybrid — you inherit node types AND compose via children. Unity is pure composition via components.

---

## Key Takeaways

1. **Assemble, don't inherit** — build entities from components
2. **Single responsibility** — one component, one job
3. **Reuse through composition** — same Health on player, enemy, barrel
4. **Interfaces for contracts** — `IDamageable`, `IInteractable`
5. **70/30 rule** — mostly reusable, some glue
6. **Don't over-split** — group related behavior sensibly