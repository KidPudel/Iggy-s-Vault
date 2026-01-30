# ScriptableObjects

> [!quote] The Core Insight ScriptableObjects store data outside of scenes. They are assets, not scene objects.

---

## What ScriptableObjects Are

A ScriptableObject is a **data container** saved as an asset (`.asset` file).

Unlike MonoBehaviours ([[MonoBehaviour Lifecycle]]):

- Not attached to GameObjects
- Not in scenes
- Exists in Project as an asset
- Shared across scenes
- One instance in memory (when loaded)

```
MonoBehaviour                    ScriptableObject
├── Lives on GameObject          ├── Lives in Project (Assets/)
├── Per-scene instance           ├── Shared asset
├── In scene hierarchy           ├── Not in hierarchy
└── Destroyed with GameObject    └── Persists in project
```

Similar to Godot Resources (`.tres`), but with important behavioral differences.

---

## Creating a ScriptableObject

### Define the Class

```csharp
[CreateAssetMenu(fileName = "NewItem", menuName = "Game/ItemData")]
public class ItemData : ScriptableObject
{
    [field: SerializeField] public string Id { get; private set; }
    [field: SerializeField] public string DisplayName { get; private set; }
    [field: SerializeField] public Sprite Icon { get; private set; }
    [field: SerializeField] public int MaxStack { get; private set; } = 99;
    [field: SerializeField] public int Value { get; private set; }
}
```

`[CreateAssetMenu]` adds it to the right-click Create menu.

### Create Asset Instances

Right-click in Project → Create → Game → ItemData

Save as: `Assets/Data/Items/Sword.asset`

---

## Using ScriptableObjects

### Reference in MonoBehaviour

```csharp
public class ItemPickup : MonoBehaviour
{
    [SerializeField] private ItemData _itemData;  // Assign in Inspector
    
    public void OnPickup()
    {
        Debug.Log($"Picked up {_itemData.DisplayName}");
        Inventory.Add(_itemData);
    }
}
```

Multiple pickups can reference the **same** `ItemData` asset.

### Reference in Other ScriptableObjects

```csharp
[CreateAssetMenu(fileName = "NewRecipe", menuName = "Game/Recipe")]
public class Recipe : ScriptableObject
{
    [SerializeField] private ItemData[] _ingredients;
    [SerializeField] private ItemData _result;
    [SerializeField] private int _resultAmount;
}
```

---

## Critical Behavior: Shared Instance

When loaded, a ScriptableObject is a **single shared instance** in memory.

```csharp
// These both point to the SAME object in memory
[SerializeField] private ItemData _sword;  // On object A
[SerializeField] private ItemData _sword;  // On object B

// Modify one → both see the change
_sword.runtimeValue = 999;  // BAD: affects everyone
```

> [!danger] Never Modify ScriptableObjects at Runtime In Editor: runtime changes **persist** after exiting Play Mode (because it's an asset). In Build: changes are lost on restart. Either way, you get inconsistent behavior. Treat ScriptableObjects as **read-only** at runtime.

---

## Data vs State Pattern

For mutable data, separate **Definition** (ScriptableObject) from **Instance** (plain C# class).

```csharp
// Definition — immutable template (ScriptableObject)
[CreateAssetMenu(menuName = "Game/ItemData")]
public class ItemData : ScriptableObject
{
    public string id;
    public string displayName;
    public int maxDurability;
}

// Instance — mutable runtime state (plain C# class)
[System.Serializable]
public class ItemInstance
{
    public string dataId;
    public int currentDurability;
    public int quantity;
    
    [System.NonSerialized]
    private ItemData _cachedData;
    
    public ItemData Data => _cachedData ??= ItemDatabase.Get(dataId);
    
    public ItemInstance(ItemData data, int qty = 1)
    {
        dataId = data.id;
        _cachedData = data;
        currentDurability = data.maxDurability;
        quantity = qty;
    }
}
```

See [[Data-Driven Design in Unity]] for the full pattern.

---

## Common Patterns

### 1. Configuration Data

Static game data that designers edit:

```csharp
[CreateAssetMenu(menuName = "Game/EnemyConfig")]
public class EnemyConfig : ScriptableObject
{
    public int maxHealth;
    public float moveSpeed;
    public float attackDamage;
    public float attackCooldown;
}
```

```csharp
public class Enemy : MonoBehaviour
{
    [SerializeField] private EnemyConfig _config;
    
    private int _currentHealth;
    
    private void Start()
    {
        _currentHealth = _config.maxHealth;  // Read from config
    }
}
```

### 2. Database / Registry

Central lookup for all items:

```csharp
[CreateAssetMenu(menuName = "Game/ItemDatabase")]
public class ItemDatabase : ScriptableObject
{
    [SerializeField] private ItemData[] _allItems;
    
    private Dictionary<string, ItemData> _lookup;
    
    public void Initialize()
    {
        _lookup = _allItems.ToDictionary(item => item.id);
    }
    
    public ItemData Get(string id) => _lookup.GetValueOrDefault(id);
}
```

### 3. Event Channels

Decoupled communication between systems:

```csharp
[CreateAssetMenu(menuName = "Events/Void Event")]
public class VoidEvent : ScriptableObject
{
    private Action _listeners;
    
    public void Register(Action listener) => _listeners += listener;
    public void Unregister(Action listener) => _listeners -= listener;
    public void Raise() => _listeners?.Invoke();
}
```

```csharp
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
    [SerializeField] private IntEvent _onHealthChanged;
    [SerializeField] private VoidEvent _onPlayerDied;
    
    public void TakeDamage(int amount)
    {
        _health -= amount;
        _onHealthChanged.Raise(_health);
        
        if (_health <= 0)
            _onPlayerDied.Raise();
    }
}

// Subscriber
public class HealthUI : MonoBehaviour
{
    [SerializeField] private IntEvent _onHealthChanged;
    
    private void OnEnable() => _onHealthChanged.Register(UpdateDisplay);
    private void OnDisable() => _onHealthChanged.Unregister(UpdateDisplay);
    
    private void UpdateDisplay(int health) => _text.text = health.ToString();
}
```

**Why this works:** Both reference the same ScriptableObject asset. No direct dependency between Player and HealthUI.

### 4. Runtime Sets

Track all active instances of something:

```csharp
[CreateAssetMenu(menuName = "Game/Runtime Set")]
public class EnemyRuntimeSet : ScriptableObject
{
    private List<Enemy> _items = new();
    
    public IReadOnlyList<Enemy> Items => _items;
    public int Count => _items.Count;
    
    public void Add(Enemy item) => _items.Add(item);
    public void Remove(Enemy item) => _items.Remove(item);
    
    private void OnEnable() => _items.Clear();  // Reset on play
}
```

```csharp
public class Enemy : MonoBehaviour
{
    [SerializeField] private EnemyRuntimeSet _enemySet;
    
    private void OnEnable() => _enemySet.Add(this);
    private void OnDisable() => _enemySet.Remove(this);
}
```

Now anything can query `_enemySet.Items` without finding objects.

### 5. Shared Variables

A single value accessible anywhere:

```csharp
[CreateAssetMenu(menuName = "Variables/Int Variable")]
public class IntVariable : ScriptableObject
{
    public int value;
    
    // Optional: event when changed
    public event Action<int> OnChanged;
    
    public void SetValue(int newValue)
    {
        value = newValue;
        OnChanged?.Invoke(value);
    }
}
```

> [!warning] Use Sparingly This is essentially a global variable. Good for things like "player score" that many systems read. Don't overuse.

---

## ScriptableObject Lifecycle

```csharp
public class MyData : ScriptableObject
{
    private void OnEnable()
    {
        // Called when asset is loaded
        // Good place to initialize runtime collections
    }
    
    private void OnDisable()
    {
        // Called when asset is unloaded
    }
    
    private void OnValidate()
    {
        // Editor only: called when values change in Inspector
    }
}
```

---

## Godot Comparison

| Godot Resource                     | Unity ScriptableObject                                                   |
| ---------------------------------- | ------------------------------------------------------------------------ |
| `.tres` file                       | `.asset` file                                                            |
| `extends Resource`                 | `: ScriptableObject`                                                     |
| `@export`                          | `[SerializeField]` or `[field: SerializeField]`                          |
| `load("res://...")`                | `Resources.Load<T>()` or Inspector reference                             |
| Cached when loaded                 | Cached when loaded                                                       |
| Can modify at runtime (be careful) | Can modify but **don't** (persists (0nly) in Editor because of the hook) |

**Key difference:** In Godot, modifying a loaded Resource at runtime is a common pattern (with care). In Unity, modifying ScriptableObjects at runtime causes Editor persistence issues. Use plain C# classes for mutable state.

> [!note] Unity's Editor has automatic serialization hooks that write changes back to disk immediately when you modify the object through the Inspector or during play mode.

---

## When to Use ScriptableObjects

**Use for:**

- Game configuration (enemy stats, item definitions)
- Shared data across scenes
- Event channels (decoupled communication)
- Databases/registries
- Designer-editable data

**Don't use for:**

- Per-instance mutable state (use plain C# class)
- Anything that changes at runtime
- Save data (use JSON/binary to `persistentDataPath`)

---

## Key Takeaways

1. **ScriptableObjects are assets** — not scene objects
2. **Shared instance in memory** — all references point to same object
3. **Never modify at runtime** — use for read-only data
4. **Data vs State** — SO for definition, plain C# for instance
5. **Event channels** — powerful decoupling pattern
6. **Designer-friendly** — editable in Inspector without code