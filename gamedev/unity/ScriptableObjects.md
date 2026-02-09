# ScriptableObjects

ScriptableObjects are data containers saved as project assets — not scene objects. They're Unity's tool for separating data from prefabs. **You don't always need them.** If your game has a handful of enemy types, putting stats directly on prefab components is simpler and perfectly fine. SOs become valuable when you have many variations, need data without instantiation, or want designers editing values separately from prefabs.

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

**The rule for data SOs:** Read-only at runtime. For mutable per-instance state, use plain C# classes. If you reach a point where you need formal Data/State separation, see [[Data-Driven Design]].

---

# ScriptableObject + Prefab Relationship

SOs hold **data** (what/rules). Prefabs hold **scene objects** (where/visuals). They connect in two directions.

## Catalog: SO References Prefabs

The SO acts as a database — it knows about things that could be created. A manager reads the catalog and decides what to instantiate.

```csharp
[CreateAssetMenu(menuName = "Game/EnemyDatabase")]
public class EnemyDatabase : ScriptableObject
{
    public EnemyEntry[] enemies;
}

[System.Serializable]
public class EnemyEntry
{
    public EnemyData data;          // another SO with stats
    public GameObject prefab;       // the prefab to spawn
}
```

**Use when:** A system needs to pick from many possible things to create — spawn managers, loot tables, shop inventories.

## Identity: Prefab References SO

The SO acts as an identity card — it tells the object what it is. The prefab's component references the SO in the Inspector.

```csharp
public class Enemy : MonoBehaviour
{
    [SerializeField] private EnemyData _data;  // assigned in prefab Inspector

    private int _currentHealth;

    private void Start()
    {
        _currentHealth = _data.MaxHealth;  // Read template values
    }
}
```

Multiple instances of the same prefab share one SO. Change the SO → all instances read the new values.

**Use when:** An object needs to know its own data (stats, configuration).

## Both Together

Common in real projects:

```
EnemyDatabase (SO) — catalog, used by SpawnManager
├── references GoblinData (SO) + Goblin (Prefab)
├── references DragonData (SO) + Dragon (Prefab)
└── references SkeletonData (SO) + Skeleton (Prefab)

Goblin (Prefab) — identity, knows it's a goblin
└── Enemy component → references GoblinData (SO)
```

| Question | Pattern |
|---|---|
| System needs to create/manage things? | SO references Prefabs (Catalog) |
| Object needs to know what it is? | Prefab references SO (Identity) |
| Both? | Manager picks from catalog → spawns prefab → prefab knows its data |

> [!tip] Ask: **who needs to know?** A system that creates things needs a catalog. An object that exists needs an identity. Often both.

---

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
3. **SO = data (what/rules), Prefab = scene object (where/visuals)**
4. **Catalog pattern:** SO references Prefabs — for systems that create things
5. **Identity pattern:** Prefab references SO — for objects that need to know what they are
6. **`[CreateAssetMenu]`** makes them designer-friendly — editable in Inspector, zero code

---

When you outgrow values-on-prefabs, see [[Data-Driven Design]] for the full Data/State/Representation pattern.
