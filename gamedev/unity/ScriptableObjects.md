# ScriptableObjects

ScriptableObjects are data containers saved as project assets — not scene objects. They're Unity's tool for storing data as reusable asset files.

---

# What They Are

A ScriptableObject is a data container that lives as an `.asset` file in your project. Not in scenes, not on GameObjects.

```
MonoBehaviour                    ScriptableObject
├── Lives on GameObject          ├── Lives in Project (Assets/)
├── Per-scene instance           ├── Shared asset
├── In scene hierarchy           ├── Not in hierarchy
└── Destroyed with GameObject    └── Persists in project
```

**The key behavior:** When loaded, a ScriptableObject is a **single shared instance** in memory. All references point to the same object. This is what makes them work as data templates — one definition, many readers.

---

# Creating ScriptableObjects

## Define the Class

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

## Create Assets

Right-click in Project → Create → Game → ItemData
Save as: `Assets/Data/Items/Sword.asset`

Fill in values in the Inspector. Multiple objects can reference the same asset.

## Reference from MonoBehaviours or Other SOs

```csharp
public class ItemPickup : MonoBehaviour
{
    [SerializeField] private ItemData _itemData;  // Assign in Inspector
}
```

```csharp
[CreateAssetMenu(menuName = "Game/Recipe")]
public class Recipe : ScriptableObject
{
    [SerializeField] private ItemData[] _ingredients;
    [SerializeField] private ItemData _result;
}
```

SOs can hold references to any asset type — prefabs, meshes, materials, sprites, other SOs. A reference is just a pointer, not a copy.

---

# Don't Modify Data SOs at Runtime

Because all references share one instance:

```csharp
// On object A and object B — same memory
[SerializeField] private ItemData _sword;

_sword.Value = 999;  // BAD: every reference now sees 999
```

**In Editor:** Changes during Play Mode persist to the `.asset` file on disk. Unity's serialization hooks write them back automatically. You'll corrupt your data without realizing it.

**In Build:** Changes exist only in RAM and are lost on restart. But they still affect every reference during that session.

**The rule for data SOs:** Read-only at runtime. For mutable per-instance state, use plain C# classes. See [[Data-Driven Design]] for the Data/State/Representation pattern.

---

# ScriptableObject + Prefab Relationship

SOs and Prefabs do different things. SOs store data (values, rules). Prefabs store structure (components, hierarchy, visuals). Two ways they connect:

## Catalog: SO Holds References to Prefabs

SO acts as a database of things a system can create.

```csharp
[CreateAssetMenu(menuName = "Game/EnemyDatabase")]
public class EnemyDatabase : ScriptableObject
{
    public GameObject[] enemyPrefabs;
}
```

A spawn manager, loot table, or shop reads the catalog and picks what to instantiate. The SO doesn't care what's inside the prefabs — it just knows they exist.

## Identity: Prefab Holds a Reference to an SO

A component on the prefab holds a reference to a data SO. The prefab knows its own data.

```csharp
public class Enemy : MonoBehaviour
{
    [SerializeField] private EnemyData _data;

    private int _currentHealth;

    private void Start()
    {
        _currentHealth = _data.MaxHealth;  // Read from the SO
    }
}
```

The reference can be assigned in the Inspector, passed through code, or set any other way — it just depends 0n amount of prefabs. The result is the same: the component reads from a shared SO instance. Multiple prefab instances referencing the same SO all read the same values.

## They're Independent

Catalog is about a system knowing what it can create. Identity is about an object knowing its own data. A prefab in a catalog can also have Identity. A prefab with Identity doesn't need to be in a catalog. Use either or both — they solve different problems.

# Lifecycle

```csharp
public class MyData : ScriptableObject
{
    private void OnEnable()
    {
        // Called when asset is loaded into memory
        // Good place to initialize runtime collections
    }

    private void OnDisable()
    {
        // Called when asset is unloaded
    }

    private void OnValidate()
    {
        // Editor only: called when values change in Inspector
        // Useful for validation and auto-filling derived fields
    }
}
```

---

# Key Takeaways

1. **SOs are shared assets** — all references point to one instance in memory
2. **Don't modify data SOs at runtime** — use plain C# classes for mutable state
3. **SO = data (values, rules), Prefab = structure (components, hierarchy, visuals)**
4. **Catalog** — SO holds prefab references so a system knows what it can create
5. **Identity** — prefab holds an SO reference so it knows its own data
6. **They're independent** — use either or both
7. **`[CreateAssetMenu]`** makes them designer-friendly — editable in Inspector, zero code

---

See [[Data-Driven Design]] for the Data/State/Representation pattern.
