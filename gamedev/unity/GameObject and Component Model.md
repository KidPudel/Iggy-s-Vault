# GameObject & Component Model

> [!quote] The Core Insight A GameObject is nothing. Components make it something.

---

## The Paradigm

Unity uses **composition over inheritance**. You don't create an "Enemy" class that inherits from a base entity. You assemble an Enemy from parts.

```
Godot thinking:
    Enemy extends CharacterBody3D
        └── inherits physics, collision
        └── your script adds behavior

Unity thinking:
    GameObject "Enemy"
        ├── Rigidbody (physics)
        ├── CapsuleCollider (collision)  
        ├── EnemyMovement (your script)
        ├── EnemyHealth (your script)
        └── EnemyAI (your script)
```

**The shift:** Stop asking "what class should this inherit from?" Start asking "what components does this need?"

---

## The Fundamental Rule

**A Component cannot have its own Transform (position/rotation/scale). Only GameObjects have Transforms.**

This single rule determines when you add a component vs when you create a child GameObject.

```
"I need to add X to my Player"
            │
            ▼
   Does X need its own position/rotation
   that's DIFFERENT from the Player's?
            │
     ┌──────┴──────┐
    YES           NO
     │             │
     ▼             ▼
  Child        Component
  GameObject   on Player
```

### Examples

|Thing|Needs own position?|Solution|
|---|---|---|
|Health|No (it's data)|Component|
|Movement script|No (moves the Player)|Component|
|Collider|No (at Player's position)|Component|
|Camera|Yes (offset behind player)|Child GameObject|
|Weapon in hand|Yes (at hand position)|Child GameObject|
|Footstep particles|Yes (at feet, not center)|Child GameObject|

### Applied to a Player

```
Player (GameObject)
├── MeshRenderer      ← same position as Player → Component
├── Rigidbody         ← same position as Player → Component
├── CapsuleCollider   ← same position as Player → Component
├── Health            ← no position (just data) → Component
├── PlayerController  ← no position (just logic) → Component
│
├── CameraRig (GameObject) [localPosition: 0, 2, -5]
│   └── Camera        ← DIFFERENT position (behind/above) → Child GameObject
│
└── WeaponSlot (GameObject) [localPosition: 0.5, 1.2, 0.3]
    └── Sword (GameObject)  ← DIFFERENT position (at hand) → Child GameObject
        ├── MeshRenderer
        └── WeaponStats
```

### Why Godot Feels Different

In Godot, **every Node has a Transform** — even nodes that don't need one. You add a Health node? It has a transform (you just ignore it).

Unity separates the concepts: **GameObjects have Transforms, Components don't.** This is more explicit about what has spatial meaning.

**Think of it this way:**

- **Components** = aspects/properties of a GameObject (health, physics, rendering)
- **Child GameObjects** = separate parts with their own positions (camera, weapons, attachments)

---

## GameObject

A GameObject is an empty container. It has:

- A name
- A Transform (always, cannot remove)
- A list of Components

That's it. A GameObject with no components (except Transform) does nothing and displays nothing.

```csharp
// Creating an empty GameObject
var go = new GameObject("MyObject");
// It now exists in the scene with only a Transform
```

**GameObjects are not typed.** There's no "EnemyObject" class. An enemy is just a GameObject with enemy-related components attached.

---

## Component

A Component is a piece of functionality attached to a GameObject. Unity provides many:

|Component|Purpose|
|---|---|
|Transform|Position, rotation, scale (always present)|
|MeshRenderer|Displays 3D mesh|
|SpriteRenderer|Displays 2D sprite|
|Rigidbody|Physics simulation|
|Collider|Collision detection|
|AudioSource|Plays sound|
|Camera|Renders the scene|

Your scripts are also components — they inherit from `MonoBehaviour`.

```csharp
public class EnemyHealth : MonoBehaviour
{
    [SerializeField] private int _maxHealth = 100;
    private int _currentHealth;
    
    private void Start()
    {
        _currentHealth = _maxHealth;
    }
    
    public void TakeDamage(int amount)
    {
        _currentHealth -= amount;
        if (_currentHealth <= 0)
            Destroy(gameObject);
    }
}
```

---

## Assembling Entities

An entity's capabilities come from its components, not its class hierarchy.

**Example: Player vs Enemy**

Both need movement and health. In inheritance thinking, you'd create a base class. In Unity:

```
GameObject "Player"                 GameObject "Enemy"
├── Rigidbody                       ├── Rigidbody
├── CapsuleCollider                 ├── CapsuleCollider
├── Health         ←── same         ├── Health         ←── same
├── PlayerInput    ←── different    ├── EnemyAI        ←── different
└── PlayerCamera                    └── (nothing)
```

The `Health` component is identical on both. You write it once. The difference is Player has `PlayerInput`, Enemy has `EnemyAI`.

**Want a destructible barrel?** Add `Health` component to a barrel. Done.

---

## Getting Components

Components on the same GameObject find each other with `GetComponent<T>()`.

```csharp
public class EnemyAI : MonoBehaviour
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

> [!warning] Cache GetComponent Results `GetComponent` is not expensive anymore (since Unity 2020.2), but calling it every frame is wasteful. Cache in `Awake()` or `Start()`.

**Requiring components:**

```csharp
[RequireComponent(typeof(Rigidbody))]
public class Movement : MonoBehaviour
{
    // Unity will auto-add Rigidbody if missing
    // and prevent its removal while this component exists
}
```

---

## Adding/Removing at Runtime

Components can be added and removed dynamically:

```csharp
// Add component
var health = gameObject.AddComponent<Health>();

// Remove component
Destroy(GetComponent<Health>());

// Check existence
if (TryGetComponent<Health>(out var health))
{
    health.TakeDamage(10);
}
```

This enables dynamic behavior changes — something inheritance hierarchies can't do easily.

---

## The Transform

Every GameObject has exactly one Transform. It defines:

- Position (`transform.position`, `transform.localPosition`)
- Rotation (`transform.rotation`, `transform.localRotation`)
- Scale (`transform.localScale`)
- Parent-child hierarchy (`transform.parent`, `transform.SetParent()`)

```csharp
// World space
transform.position = new Vector3(1, 2, 3);

// Local to parent
transform.localPosition = Vector3.zero;

// Parenting
transform.SetParent(otherTransform);
```

See [[Transform and Hierarchy]] for details.

---

## Mental Model

|Concept|What It Is|
|---|---|
|GameObject|Empty container, identity only|
|Component|One piece of behavior or data|
|MonoBehaviour|Base class for your script components|
|Transform|Spatial data, always present|
|Prefab|Saved GameObject template (see [[Prefabs]])|

**Analogy:**

- GameObject = Actor on stage (has a name, a position)
- Components = Costumes, props, scripts the actor uses
- The actor isn't defined by inheritance — they're defined by what they're wearing and holding

---

## Godot Comparison

|Godot|Unity|Note|
|---|---|---|
|Node|GameObject|Container|
|Script on Node|MonoBehaviour component|Behavior|
|Node inheritance (`extends CharacterBody3D`)|Component composition|Different paradigm|
|`get_node()`|`transform.Find()` or serialized reference|Unity prefers Inspector references|
|`$Child`|`transform.Find("Child")`|Or `[SerializeField]`|

The key difference: In Godot, you pick a node type and extend it. In Unity, you start with an empty GameObject and add what you need.

---

## When to Use Multiple Components vs One

**Split when:**

- Functionality is reusable (Health on player, enemy, barrel)
- You want to enable/disable parts independently
- Different team members work on different aspects

**Keep together when:**

- Logic is tightly coupled and won't be reused
- Splitting creates excessive cross-component communication

Jonas Tyroller's 70/30 rule applies: ~70% reusable isolated components, ~30% glue code that's specific to this entity.

---

## Key Takeaways

1. **GameObjects are empty** — components define behavior
2. **Composition over inheritance** — assemble, don't extend
3. **One component, one responsibility** — reusable pieces
4. **Cache component references** — get them in Awake/Start
5. **Dynamic assembly** — add/remove components at runtime

> [!quote] The Test When you want to add a feature to an entity, your first thought should be "what component does it need?" not "what class should it inherit from?"