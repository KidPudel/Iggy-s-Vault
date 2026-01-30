# Data-Driven Design in Unity

> [!quote] The Core Insight Your game's items, enemies, quests — that IS the game. Code is just an interpreter.

---

## The Problem

Without data-driven design:

```
"I need to code an iron axe"
→ IronAxeController.cs with hardcoded stats
→ Copy-paste for SteelAxe.cs, MithrilAxe.cs...
→ Designer wants to tweak damage
→ Edit 47 files
```

With data-driven design:

```
"What data describes an axe?"
→ Define shape (ScriptableObject class)
→ Create IronAxe.asset, SteelAxe.asset...
→ One script interprets any axe
→ Designer tweaks in Inspector, zero code changes
```

**The shift:** Stop thinking "code X." Start thinking "what data describes X?"

---

## Three Layers

|Layer|Purpose|Unity Implementation|
|---|---|---|
|**Data**|Template, "platonic ideal"|ScriptableObject|
|**State**|Runtime instance, mutable|Plain C# class|
|**Representation**|Visual/interactive form|MonoBehaviour + GameObject|

```
DATA (IronAxe.asset):           STATE (player's axe):        REPRESENTATION:
├─ id: "iron_axe"               ├─ dataId: "iron_axe"        ├─ MeshRenderer
├─ displayName: "Iron Axe"      ├─ durability: 47            ├─ Collider
├─ maxDurability: 100           └─ enchantments: [fire]      └─ WorldItem.cs
├─ damage: 25
└─ icon: axe.png
```

---

## Data Layer (ScriptableObject)

Defines what something CAN be. **Read-only at runtime.**

```csharp
[CreateAssetMenu(menuName = "Game/ItemData")]
public class ItemData : ScriptableObject
{
    [field: SerializeField] public string Id { get; private set; }
    [field: SerializeField] public string DisplayName { get; private set; }
    [field: SerializeField] public Sprite Icon { get; private set; }
    [field: SerializeField] public int MaxDurability { get; private set; } = 100;
    [field: SerializeField] public int MaxStack { get; private set; } = 1;
    [field: SerializeField] public int Damage { get; private set; } = 10;
}
```

Create assets: Right-click → Create → Game → ItemData Save as: `Assets/Data/Items/IronAxe.asset`

See [[ScriptableObjects]] for details.

---

## State Layer (Plain C# Class)

Represents what something IS now. **Mutable.**

```csharp
[System.Serializable]
public class ItemInstance
{
    public string dataId;
    public int durability;
    public int quantity;
    public List<string> enchantments = new();
    
    [System.NonSerialized] private ItemData _cachedData;
    
    public ItemData Data => _cachedData ??= ItemDatabase.Get(dataId);
    
    public ItemInstance() { }  // For deserialization
    
    public ItemInstance(ItemData data, int qty = 1)
    {
        dataId = data.Id;
        _cachedData = data;
        durability = data.MaxDurability;
        quantity = qty;
    }
}
```

**Key points:**

- References Data by ID (not direct reference) for serialization
- `[System.Serializable]` for JSON save/load
- Parameterless constructor for deserialization
- Caches Data reference for runtime access

---

## Representation Layer (MonoBehaviour)

Visual/interactive form. Can be destroyed/recreated without losing state.

```csharp
public class WorldItem : MonoBehaviour
{
    private ItemInstance _instance;
    
    [SerializeField] private SpriteRenderer _renderer;
    [SerializeField] private TextMeshPro _label;
    
    public void Setup(ItemInstance instance)
    {
        _instance = instance;
        _renderer.sprite = instance.Data.Icon;
        _label.text = instance.Data.DisplayName;
    }
    
    public void OnInteract()
    {
        if (Inventory.TryAdd(_instance))
        {
            Destroy(gameObject);  // GameObject dies, instance lives in inventory
        }
    }
}
```

---

## Where They Live

|Layer|At Runtime|On Disk|
|---|---|---|
|Data|Cached asset (shared)|`Assets/Data/Items/IronAxe.asset`|
|State|RAM objects (unique)|`persistentDataPath/save.json`|
|Representation|GameObjects (scene)|Not saved (recreated from State)|

---

## When to Use What

```
Does data need to outlive the GameObject?
    │
    ├─ NO  → Data only (MonoBehaviour holds state)
    │        Enemy dies = data gone. That's fine.
    │
    └─ YES → Data + State
             Item picked up = GameObject destroyed, state lives.
             
             Does it need visual representation?
                 │
                 ├─ YES → Data + State + Representation
                 │        Item on ground, in inventory, equipped
                 │
                 └─ NO  → Data + State only
                          Quest progress, player stats
```

### Quick Reference

|Situation|Pattern|
|---|---|
|Enemy (dies permanently)|Data only, state on component|
|Bullet, particle|Data only|
|Inventory item|Data + State + Representation|
|Player stats|State only|
|Quest progress|State only|
|Placeable furniture|Data + State + Representation|

---

## Database Pattern

Central lookup for all data:

```csharp
[CreateAssetMenu(menuName = "Game/ItemDatabase")]
public class ItemDatabase : ScriptableObject
{
    [SerializeField] private ItemData[] _items;
    
    private Dictionary<string, ItemData> _lookup;
    
    public void Initialize()
    {
        _lookup = _items.ToDictionary(i => i.Id);
    }
    
    public ItemData Get(string id) => _lookup.GetValueOrDefault(id);
    
    public ItemInstance CreateInstance(string id, int qty = 1)
    {
        var data = Get(id);
        return data != null ? new ItemInstance(data, qty) : null;
    }
}
```

---

## Saving and Loading

### Save Structure

```csharp
[System.Serializable]
public class SaveData
{
    public Vector3 playerPosition;
    public List<ItemInstance> inventory = new();
    public Dictionary<string, QuestState> quests = new();
}
```

### Save/Load

```csharp
public void Save(string slot)
{
    var data = new SaveData
    {
        playerPosition = _player.transform.position,
        inventory = _inventory.Items.ToList(),
        quests = _questManager.GetStates()
    };
    
    string json = JsonUtility.ToJson(data);
    string path = Path.Combine(Application.persistentDataPath, $"save_{slot}.json");
    File.WriteAllText(path, json);
}

public SaveData Load(string slot)
{
    string path = Path.Combine(Application.persistentDataPath, $"save_{slot}.json");
    if (!File.Exists(path)) return null;
    
    string json = File.ReadAllText(path);
    return JsonUtility.FromJson<SaveData>(json);
}
```

> [!warning] JsonUtility Limitations `JsonUtility` doesn't support Dictionary. Use Newtonsoft.Json or convert to List of key-value pairs.

**What to save:** State layer only (current values, progress). **What NOT to save:** Data layer (already in Assets), Representation (recreated).

---

## Common Patterns

### Spawning

```csharp
public void SpawnItem(string id, Vector3 position)
{
    var instance = _database.CreateInstance(id);
    var go = Instantiate(_worldItemPrefab, position, Quaternion.identity);
    go.GetComponent<WorldItem>().Setup(instance);
}
```

### Picking Up (State survives, GameObject dies)

```csharp
public void OnInteract()
{
    if (Inventory.TryAdd(_instance))
    {
        Destroy(gameObject);  // State lives in inventory
    }
}
```

### Dropping (State gets new Representation)

```csharp
public void Drop(ItemInstance instance, Vector3 position)
{
    Inventory.Remove(instance);
    var go = Instantiate(_worldItemPrefab, position, Quaternion.identity);
    go.GetComponent<WorldItem>().Setup(instance);  // Same state, new GO
}
```

---

## Extending Data Types

### Inheritance

```csharp
[CreateAssetMenu(menuName = "Game/WeaponData")]
public class WeaponData : ItemData
{
    [field: SerializeField] public int AttackDamage { get; private set; }
    [field: SerializeField] public float AttackSpeed { get; private set; }
}
```

**Pros:** Type-safe, clear hierarchy. **Cons:** Deep hierarchies get messy.

### Composition

```csharp
[CreateAssetMenu(menuName = "Game/ItemData")]
public class ItemData : ScriptableObject
{
    [field: SerializeField] public string Id { get; private set; }
    [field: SerializeField] public WeaponComponent Weapon { get; private set; }
    [field: SerializeField] public ConsumableComponent Consumable { get; private set; }
}

[System.Serializable]
public class WeaponComponent
{
    public int damage;
    public float attackSpeed;
}
```

**Pros:** Flexible, items can have multiple aspects. **Cons:** Null checks, less type safety.

---

## Common Pitfalls

|Smell|Problem|Fix|
|---|---|---|
|`itemData.damage = 999`|Mutates shared asset|Never modify Data at runtime|
|SO reference in save file|Breaks if asset moved/renamed|Use string ID, resolve at load|
|Logic in ScriptableObject|Hard to test|Keep logic in Systems|
|State on MonoBehaviour only|Lost when destroyed|Separate State class|

---

## Key Takeaways

1. **Data** (ScriptableObject) = template, read-only
2. **State** (plain C# class) = runtime values, mutable
3. **Representation** (MonoBehaviour) = visual, disposable
4. **Reference by ID** for serialization
5. **Save State only** — Data is in Assets, Representation is recreated
6. **Design data first** — understand your structures before coding

> [!quote] The Test Can someone create a new item by ONLY creating a ScriptableObject asset?

If yes, you've achieved data-driven design.