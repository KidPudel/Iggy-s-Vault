# GameObject & Component Model

> [!quote] The Core Insight
> A GameObject is nothing. Components make it something.

---

## The Paradigm

Unity uses **composition over inheritance**. You don't create an "Enemy" class that inherits from a base entity. You assemble an Enemy from parts.
More on that here [[Component-Based Design in Unity]]

```
Godot thinking:
    Enemy extends CharacterBody3D
        └── inherits physics, collision automatically
        └── your script adds behavior

Unity thinking:
    GameObject "Enemy"
        ├── Rigidbody (physics)
        ├── CapsuleCollider (collision)  
        ├── EnemyMovement (your script)
        └── EnemyHealth (your script)
```

**The shift:** Stop asking "what class should this inherit from?" Start asking "what components does this need?"

---

## The Fundamental Rule

**A Component cannot have its own Transform. Only GameObjects can.**

This single rule determines when you add a component vs. when you create a child GameObject.

```
"I need to add X to my Player"
            │
            ▼
   Does X need its own position/rotation
   that's DIFFERENT from the Player's logic?
            │
     ┌──────┴──────┐
    YES           NO
     │             │
     ▼             ▼
  Child        Component
  GameObject   on Player
```

### Decision Matrix

| Feature | Needs own Transform? | Solution |
| :--- | :--- | :--- |
| **Health** | No (pure data) | `Component` |
| **Movement Logic** | No (moves the root) | `Component` |
| **Main Hitbox** | No (usually matches root) | `Component` |
| **Model/Sprite** | **YES** (visuals often offset/anim) | **Child GameObject** |
| **Weapon Hand** | **YES** (specific offset) | **Child GameObject** |
| **Camera Follow** | **YES** (offset position) | **Child GameObject** |

---

## Best Practice Hierarchy: The "Logic vs. Visuals" Split

A common mistake is putting the `MeshRenderer` or `SpriteRenderer` on the root object. **Don't do this.**

**The CodeMonkey / Pro Standard:**
Separate your **Logic Root** from your **Visual Representation**.

```
Player (GameObject)           <-- THE LOGIC ROOT
├── Rigidbody                 (Physics lives here)
├── CapsuleCollider           (Collision lives here)
├── PlayerController          (Scripts live here)
│
├── Visuals (GameObject)      <-- THE LOOKS
│   ├── MeshRenderer          (The actual model)
│   └── Animator              (Animations affect this transform)
│
├── CameraTarget (GameObject) <-- LOGICAL ANCHOR
│   (Empty object defining where camera looks)
│
└── WeaponHolder (GameObject) <-- PHYSICAL ANCHOR
    └── Sword (GameObject)
        ├── MeshRenderer
        └── WeaponStats
```

**Why this matters:**
1.  **Animation Safety:** You can squash/stretch/rotate the `Visuals` child without affecting the `Collider` or `Rigidbody` on the root.
2.  **Swapping Models:** You can delete `Visuals` and replace it with a different model without breaking your movement scripts.

---

## GameObject

A GameObject is an empty container. It has:
1.  A **Name**
2.  A **Tag** & **Layer**
3.  A **Transform** (mandatory, cannot be removed)
4.  A list of **Components**

**GameObjects are not typed.** There is no "EnemyObject" class. An enemy is just a generic GameObject that happens to have `EnemyAI` attached.

---

## Component

A Component is a cohesive chunk of behavior.

**The "One Job" Rule:**
Avoid "God Components" (e.g., `PlayerManager.cs` that does input, health, and sound). Split them up!

*   `PlayerMovement.cs` (Input → Physics)
*   `PlayerHealth.cs` (Damage → Events)
*   `PlayerAnimation.cs` (State → Visuals)

### Common Built-in Components

| Component | Purpose | Godot Equivalent |
| :--- | :--- | :--- |
| **Transform** | Position, Rot, Scale | `Node3D` properties |
| **MeshRenderer** | Renders geometry | `MeshInstance3D` |
| **Rigidbody** | Physics simulation | `RigidBody3D` settings |
| **Collider** | Physical shape | `CollisionShape3D` |
| **AudioSource** | Sound player | `AudioStreamPlayer3D` |

### Script Component Template
Your scripts are components. Use `[SerializeField]` to expose private variables to the Inspector.

```csharp
public class EnemyHealth : MonoBehaviour
{
    // Best Practice: Group fields with Headers
    [Header("Settings")]
    [SerializeField] private int _maxHealth = 100;
    
    [Header("Debug")]
    [SerializeField] private int _currentHealth;
    
    // runs once when object is created (like _ready, but strictly initialization)
    private void Awake() 
    {
        _currentHealth = _maxHealth;
    }
    
    public void TakeDamage(int amount)
    {
        _currentHealth -= amount;
        if (_currentHealth <= 0)
        {
            Debug.Log("Enemy died!");
            Destroy(gameObject);
        }
    }
}
```
