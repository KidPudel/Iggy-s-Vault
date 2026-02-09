# Component-Based Design

> [!quote] The Core Insight Inheritance defines what something **IS**. Composition defines what something **CAN DO**. Game entities are better defined by capabilities than identity.

---

# The Principle

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

# Why This Matters

## The Identity Trap

Inheritance forces you to answer: "What **IS** this thing?"

That sounds reasonable until your game design starts crossing categories. A barrel that can be picked up, thrown, set on fire, and deals damage on impact — is that a `Prop`? A `Weapon`? A `Projectile`? A `DamageSource`? With inheritance, you must choose one identity and force everything else through it. The class hierarchy becomes a taxonomy, and taxonomies break when reality doesn't fit your categories.

Composition asks a different question: "What can this thing **DO**?"

A barrel HAS health. It HAS a grabbable capability. It HAS a damage-on-impact behavior. None of these are its identity — they're capabilities bolted on. Remove one, add another, and the barrel is still a barrel.

## The Single Inheritance Wall

C# only allows single inheritance, so the identity problem isn't just philosophical — it's mechanical:

```csharp
// Inheritance: pick ONE identity
class FlyingSwimmingElectricEnemy : ???
{
    // Does it inherit from FlyingEnemy, SwimmingEnemy, or ElectricEnemy?
    // C# forces you to choose. You can only have one base class.
}
```

## The Component Solution

```
// Composition: combine ANY capabilities
GameObject "WeirdCreature"
├── Health
├── FlyMovement
├── SwimMovement
├── ElectricAttack
└── EnemyAI
```

Want a non-flying version? Remove `FlyMovement`. No code changes. No new class needed. No refactoring a hierarchy.

# Single Responsibility

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

# Reusable Components

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

# Component Communication

Components on the same GameObject talk via:

## Direct Reference (GetComponent)

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

## RequireComponent

Enforce dependencies:

```csharp
[RequireComponent(typeof(Health))]
public class DeathHandler : MonoBehaviour
{
    // Health is guaranteed to exist
}
```

---

# Interfaces for Polymorphism

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

# Composition Patterns

## Behavior Assembly

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

## Dynamic Composition

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

# Two Kinds of Composition

Composition works differently depending on whether you're working with MonoBehaviours or plain C# classes. Understanding the relationship model matters.

## MonoBehaviour Composition: Peers

Components on a GameObject are **siblings**. No component owns another — the **GameObject** owns them all. Components discover each other.

```
GameObject "Enemy"
├── Health          ← not owned by EnemyAI
├── Movement        ← not owned by EnemyAI
├── EnemyAI         ← doesn't "contain" the others
└── Animator        ← just another sibling
```

```csharp
public class EnemyAI : MonoBehaviour
{
    private Health _health;  // Reference to a peer, not a child
    
    private void Awake()
    {
        // "Is there a Health sitting next to me on this GameObject?"
        _health = GetComponent<Health>();
    }
}
```

EnemyAI didn't create Health. It doesn't control Health's lifetime. If Health gets removed, EnemyAI just loses its reference. They collaborate as equals.

## Plain C# Composition: Ownership

In plain C# classes, composition means **one object contains and controls another**. Traditional parent-child ownership.

```csharp
public class CombatResolver
{
    private readonly DamageCalculator _damage;  // I own this
    private readonly ArmorSystem _armor;        // I own this too
    
    public CombatResolver()
    {
        _damage = new DamageCalculator();  // I created it
        _armor = new ArmorSystem();        // I control its lifetime
    }
    
    public int Resolve(AttackData attack, DefenseData defense)
    {
        int raw = _damage.Calculate(attack);     // I decide when to call it
        int mitigated = _armor.Reduce(raw, defense);
        return mitigated;
    }
}
```

Nothing else knows `DamageCalculator` exists. It's a private implementation detail of `CombatResolver`.

## The Relationship Difference

| MonoBehaviour    | Plain C#                                |                                       |
| :--------------- | :-------------------------------------- | :------------------------------------ |
| **Relationship** | Peers / siblings                        | Parent owns child                     |
| **Creation**     | Unity manages (Inspector, AddComponent) | You call `new`                        |
| **Discovery**    | `GetComponent<T>()` to find siblings    | You already have it (it's your field) |
| **Lifetime**     | GameObject controls it                  | You control it                        |
| **Visibility**   | Other components can see it             | Private implementation detail         |

## Mixing Both (Common and Recommended)

A single MonoBehaviour often uses both models at once:

```csharp
public class EnemyAI : MonoBehaviour
{
    // --- Sibling components (discovered, not owned) ---
    private Health _health;
    private Movement _movement;
    
    // --- Owned plain C# objects (private implementation details) ---
    private StateMachine _stateMachine;
    private PathfindingQuery _pathQuery;
    
    private void Awake()
    {
        // Discover peers
        _health = GetComponent<Health>();
        _movement = GetComponent<Movement>();
        
        // Create owned internals
        _stateMachine = new StateMachine();
        _stateMachine.AddState(new IdleState(this));
        _stateMachine.AddState(new ChaseState(this));
        
        _pathQuery = new PathfindingQuery();
    }
    
    private void Update()
    {
        // Drive my own internal systems
        _stateMachine.Update();
    }
}
```

`Health` and `Movement` are shared capabilities — other components on this GameObject might also use them. `StateMachine` and `PathfindingQuery` are internal to EnemyAI — nobody else needs to know they exist.

## When to Use Which

**Make it a MonoBehaviour component when:**

- Other components or systems need to interact with it
- It's a shared capability (multiple things might have it)
- You want to configure it in the Inspector
- You need Unity lifecycle methods (Update, collision callbacks, etc.)
- It defines what an entity **CAN DO** — a capability

**Make it a plain C# class when:**

- It's an implementation detail of one component
- It's pure logic or data with no Unity dependencies
- It's testable without Unity (just NUnit, no Play Mode)
- It doesn't need a position in the scene
- Nobody outside the owning class needs to know it exists

See [[Layered Architecture]] for the Systems/Managers/Components pattern, which relies heavily on plain C# classes for the Systems layer.

---

# The 70/30 Rule

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

# When Inheritance IS Appropriate

"Composition over inheritance" doesn't mean "never inherit." It means **don't reach for inheritance first**. There are cases where inheritance is the right tool.

## Shallow Inheritance for Shared Plumbing

When you have multiple components that genuinely share setup logic and differ only in specifics, a single level of inheritance is fine:

```csharp
// One level deep — this is fine
public abstract class Projectile : MonoBehaviour
{
    [SerializeField] protected float _speed;
    [SerializeField] protected int _damage;
    protected Rigidbody _rb;
    
    protected virtual void Awake() => _rb = GetComponent<Rigidbody>();
    
    protected virtual void OnTriggerEnter(Collider other)
    {
        if (other.TryGetComponent<IDamageable>(out var target))
            target.TakeDamage(_damage);
        
        OnImpact();
        Destroy(gameObject);
    }
    
    protected abstract void OnImpact(); // Subclass defines visual/audio
}

public class ArrowProjectile : Projectile
{
    protected override void OnImpact()
    {
        // Stick arrow into surface, play thud sound
    }
}

public class FireballProjectile : Projectile
{
    [SerializeField] private float _explosionRadius;
    
    protected override void OnImpact()
    {
        // Spawn explosion VFX, apply area damage
    }
}
```

This works because all projectiles genuinely share the "move forward, hit something, deal damage, self-destruct" pattern. The variation is small and well-defined.

## When Inheritance Breaks Down

The moment you need combinations, inheritance fails:

```csharp
// Starts fine...
public class MeleeAttack : Attack { }
public class RangedAttack : Attack { }

// Then a designer asks for...
public class MeleeAttackThatAlsoShootsProjectiles : ??? 
{
    // Uh oh
}
```

**The rule of thumb:** If you can draw the hierarchy as a clean tree that won't need cross-branches, shallow inheritance is fine. The moment you want to mix capabilities from different branches — switch to composition.

## Abstract Base vs Interface

|                   | Abstract                                          | Interface                                     |
| :---------------- | :------------------------------------------------ | :-------------------------------------------- |
| **Use when**      | Subclasses share real implementation              | Types share a contract but not implementation |
| **Carries code?** | Yes (fields, methods)                             | Minimal (default methods only)                |
| **How many?**     | One only (single inheritance)                     | As many as you want                           |
| **Example**       | `Projectile` base with shared launch/impact logic | `IDamageable` — anything that takes damage    |

Interfaces are almost always the safer choice because they don't limit your inheritance slot. Reserve abstract base classes for when you genuinely have shared implementation that would be duplicated otherwise.

---

# Anti-Patterns

## God Component

```csharp
// BAD: One component does everything
public class Player : MonoBehaviour
{
    // 2000 lines of movement, combat, inventory, UI, audio...
}
```

Split by responsibility.

## Excessive Granularity

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

## Deep Inheritance in Components

```csharp
// BAD: Deep inheritance chains
public class Attack : MonoBehaviour { }
public class MeleeAttack : Attack { }
public class SwordAttack : MeleeAttack { }
public class FireSwordAttack : SwordAttack { }
```

Each level adds coupling. Changing `Attack` ripples through three subclasses. `FireSwordAttack` inherits behaviors it might not want and can't opt out of. This is the identity trap — you've committed to a taxonomy.

**Instead:** Use a single `Attack` component configured by a ScriptableObject for data variation:

```csharp
public class Attack : MonoBehaviour
{
    [SerializeField] private AttackData _data; // ScriptableObject
}

// AttackData SO defines: damage, range, element, animation, VFX
// SwordAttackData, FireSwordAttackData — just different asset files
// No new classes needed for each weapon type
```

See [[ScriptableObjects]] for this pattern.

---

# Godot Comparison

|Godot|Unity|
|---|---|
|Node inheritance + child composition|Component composition|
|`extends CharacterBody3D`|`RequireComponent` + composition|
|Multiple scripts per node (possible)|Multiple components per GameObject (normal)|
|Scenes as reusable compositions|Prefabs as reusable compositions|

Godot is a hybrid — you inherit node types AND compose via children. Unity is pure composition via components.

Coming from Godot, the main mental shift: in Godot you think "what node type do I extend? (for base node)" In Unity you think "what components does this empty GameObject need?"

---

# Key Takeaways

1. **Capabilities, not identity** — entities are defined by what they CAN DO, not what they ARE
2. **Assemble, don't inherit** — build entities from components
3. **Single responsibility** — one component, one job
4. **Reuse through composition** — same Health on player, enemy, barrel
5. **Interfaces for contracts** — `IDamageable`, `IInteractable`
6. **Two composition models** — MonoBehaviour (peers on a GameObject) and plain C# (owned fields). Mix both.
7. **Inheritance is fine when shallow** — one level for shared plumbing, but switch to composition when you need combinations
8. **70/30 rule** — mostly reusable, some glue
9. **Don't over-split** — group related behavior sensibly