# Serialization & Inspector

> [!quote] The Core Insight Unity saves what it can see. If it's not serialized, it doesn't exist in the Inspector or saved data.

---

## What Serialization Does

Serialization converts objects to a saveable format. Unity uses it for:

- Displaying fields in Inspector
- Saving scenes and prefabs
- Storing asset data

**If a field isn't serialized, it won't appear in Inspector and won't be saved.**

---

## Serialization Rules

### What Unity Serializes by Default

| Field Type                        | Serialized?                 |
| --------------------------------- | --------------------------- |
| `public` field                    | ✅ Yes                       |
| `private` field                   | ❌ No                        |
| `private` with `[SerializeField]` | ✅ Yes                       |
| `public` with `[NonSerialized]`   | ❌ No                        |
| `static` field                    | ❌ Never                     |
| `const` field                     | ❌ Never                     |
| `readonly` field                  | ❌ Never                     |
| Property                          | ❌ No (unless backing field) |

### Serializable Types

| Type                                                                          | Serialized?                          |
| ----------------------------------------------------------------------------- | ------------------------------------ |
| Primitives (`int`, `float`, `bool`, `string`)                                 | ✅                                    |
| Unity types (`Vector3`, `Color`, `AnimationCurve`)                            | ✅                                    |
| `enum`                                                                        | ✅                                    |
| Arrays and `List<T>` of serializable types                                    | ✅                                    |
| Unity Object references (`GameObject`, `Transform`, `ScriptableObject`, etc.) | ✅                                    |
| `[System.Serializable]` classes/structs                                       | ✅                                    |
| `Dictionary<K,V>`                                                             | ❌ No                                 |
| Interfaces                                                                    | ❌ No                                 |
| Abstract classes                                                              | ❌ No (unless `[SerializeReference]`) |

---

## `[SerializeField]`

Exposes private fields to Inspector while keeping them private in code.

```csharp
public class Player : MonoBehaviour
{
    [SerializeField] private int _maxHealth = 100;
    [SerializeField] private float _moveSpeed = 5f;
    [SerializeField] private GameObject _bulletPrefab;
}
```

**This is the recommended pattern.** Keep fields private, expose via `[SerializeField]`.

---

## Properties with Serialization

Properties aren't serialized by default. Two approaches:

### Backing Field (Recommended)

```csharp
[field: SerializeField] 
public int MaxHealth { get; private set; } = 100;
```

### Explicit Backing Field

```csharp
[SerializeField] private int _maxHealth = 100;
public int MaxHealth => _maxHealth;
```

---

## Custom Serializable Classes

Mark with `[System.Serializable]` to embed in Inspector:

```csharp
[System.Serializable]
public class WeaponStats
{
    public int damage;
    public float attackSpeed;
    public float range;
}

public class Weapon : MonoBehaviour
{
    [SerializeField] private WeaponStats _stats;  // Shows as foldout in Inspector
}
```

> [!warning] No Polymorphism Basic serialization doesn't support polymorphism. A field of type `Animal` won't serialize a `Dog` subclass correctly. Use `[SerializeReference]` for that.

---

## [SerializeReference]

Enables polymorphic serialization:

```csharp
[System.Serializable]
public abstract class Action
{
    public abstract void Execute();
}

[System.Serializable]
public class MoveAction : Action
{
    public Vector3 destination;
    public override void Execute() { /* ... */ }
}

[System.Serializable]
public class WaitAction : Action
{
    public float duration;
    public override void Execute() { /* ... */ }
}

public class ActionSequence : MonoBehaviour
{
    [SerializeReference] private List<Action> _actions;  // Can hold mixed types
}
```

---

## Inspector Attributes

### Organization

```csharp
[Header("Movement")]
[SerializeField] private float _speed;
[SerializeField] private float _jumpForce;

[Space(10)]

[Header("Combat")]
[SerializeField] private int _damage;

[Tooltip("Seconds between attacks")]
[SerializeField] private float _attackCooldown;
```

### Constraints

```csharp
[Range(0, 100)]
[SerializeField] private int _health;

[Min(0)]
[SerializeField] private float _speed;

[TextArea(3, 5)]
[SerializeField] private string _description;

[Multiline(3)]
[SerializeField] private string _notes;
```

### Visibility Control

```csharp
[HideInInspector]
public int hiddenPublicField;  // Public but not in Inspector

[System.NonSerialized]
public int notSavedField;  // Not serialized at all
```

---

## Godot Comparison

|Godot|Unity|
|---|---|
|`@export var health: int`|`[SerializeField] private int _health;`|
|`@export_range(0, 100)`|`[Range(0, 100)]`|
|`@export_enum("A", "B")`|Use `enum` type|
|`@export_group("Combat")`|`[Header("Combat")]`|
|`@export_file("*.png")`|Use specific type (`Sprite`, `Texture2D`)|

---

## Common Pitfalls

|Problem|Cause|Fix|
|---|---|---|
|Field not showing in Inspector|Not serialized|Add `[SerializeField]` or make public|
|Dictionary not saving|Dictionaries not serializable|Use two parallel lists or custom serialization|
|Interface field empty|Interfaces not serializable|Use `[SerializeReference]` or concrete type|
|Values reset on play|Field not serialized|Check serialization rules|
|Changes not saving|Editing at runtime|Runtime changes don't persist (except SO in Editor)|

---

## Key Takeaways

1. **`[SerializeField]`** = private but visible in Inspector
2. **`[field: SerializeField]`** = serialized property
3. **`[System.Serializable]`** = make custom class serializable
4. **`[SerializeReference]`** = enable polymorphism
5. **Dictionaries don't serialize** — use alternatives
6. **Properties need backing fields** to serialize