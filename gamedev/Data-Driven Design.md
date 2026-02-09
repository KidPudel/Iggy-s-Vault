# Data-Driven Design

---

## Two Levels

There are two levels to this, and they solve different problems.

### Level 1: Extract values into data assets

The moment you're duplicating prefabs/scenes just to change numbers (3 enemy types with the same structure but different stats), pull those values into a data asset. The prefab reads from the data asset instead of hardcoding values.

```
BEFORE: 3 identical prefabs with different numbers
  GoblinPrefab  (Health: 50,  Speed: 3, Damage: 5)
  OrcPrefab     (Health: 120, Speed: 2, Damage: 15)
  SkeletonPrefab(Health: 80,  Speed: 4, Damage: 10)

AFTER: 1 prefab, 3 data assets
  EnemyPrefab → reads from data asset
  GoblinData.asset  (health: 50,  speed: 3, damage: 5)
  OrcData.asset     (health: 120, speed: 2, damage: 15)
  SkeletonData.asset(health: 80,  speed: 4, damage: 10)
```

This is lightweight. It's just the Identity pattern — a data asset referenced by a prefab. You don't need databases, State classes, or string IDs. Do this whenever it makes your life easier. There's no threshold to wait for.

### Level 2: Data + State + Representation (the full pattern)

This is heavier. It adds a State layer and formal separation. You need it when:

- **State outlives the visual** — item picked up, world object destroyed, state lives in inventory
- **Data without instantiation** — shop UI shows item stats before any object exists in the world
- **Saveable instance state** — each sword tracks its own durability and enchantments on disk
- **Designer workflow** — a team where designers edit data files without touching prefabs or code

If you don't have these problems, Level 1 is enough. **The rest of this doc covers Level 2.**

---

## The Full Pattern

The core idea: separate the template (what something CAN be), the instance (what it IS now), and the visual (how it appears).

### Three Layers

| Layer | Purpose | Mutability | Answers |
|---|---|---|---|
| **Data** | Template — "platonic ideal" | Read-only | "What CAN this be?" |
| **State** | Runtime instance — this specific one | Mutable | "What IS this now?" |
| **Representation** | Visual/interactive form | Disposable | "How does it appear?" |

```
DATA (iron_axe):               STATE (player's axe):        REPRESENTATION:
├─ id: "iron_axe"              ├─ dataId: "iron_axe"        ├─ Mesh
├─ displayName: "Iron Axe"     ├─ durability: 47            ├─ Collider
├─ maxDurability: 100          └─ enchantments: [fire]      └─ InteractScript
├─ damage: 25
└─ icon: axe.png
```

**Data** is a shared template — one "Iron Axe" definition, referenced by every iron axe in the game. **State** is per-instance — *this* axe has 47 durability and a fire enchantment. **Representation** is the visual that can be destroyed and recreated without losing State.

State references Data by string ID (not direct reference) so it survives serialization. When you save/load, a string ID resolves against the database. A direct object reference means nothing on disk.

### What Lives Where

A common confusion: if Data is the "template" and Representation is the "visual," where does a sprite reference belong?

**Data stores WHAT — values that vary between types.** This includes references to visual assets (sprites, meshes, sounds). "Goblins use `goblin.png`" is a fact about goblins — that's data.

**Representation is the THING that displays it.** The actual renderer component, the collider, the scripts wired together — the structure of the object.

| If it... | It lives in... |
|---|---|
| Varies between types (stats, which sprite, which mesh) | Data |
| Is the structure of how to display things (components, hierarchy, wiring) | Representation (prefab/scene) |

At runtime they meet: `renderer.sprite = data.sprite` — Representation reads Data to configure itself.

For simple games, one generic prefab serves all types — Data drives which sprite, name, stats to show. For complex cases where types need fundamentally different structures (a boss needs extra particle effects a goblin doesn't), you use different prefabs. Data can reference which prefab to use — that's the Catalog pattern.

---

### When to Use Which Layers

```
Does data need to outlive the visual object?
    │
    ├─ NO → Data only (object holds state directly)
    │       Enemy dies = data gone. That's fine.
    │
    └─ YES → Data + State
             Item picked up = visual destroyed, state lives in inventory.

             Does it need visual representation?
                 │
                 ├─ YES → Data + State + Representation
                 │        Item on ground, in inventory, equipped.
                 │
                 └─ NO  → Data + State only
                          Quest progress, player stats.
```

| Situation | Pattern |
|---|---|
| Enemy (dies permanently) | Data only, state on the object |
| Bullet, particle | Data only |
| Inventory item | Data + State + Representation |
| Player stats | State only |
| Quest progress | State only |
| Placeable furniture | Data + State + Representation |

---

### Data Layer

Defines what something CAN be. Shared by all instances. **Never modified at runtime** — mutating it corrupts every reference.

```
class ItemData:
    id: string
    displayName: string
    icon: Image
    maxDurability: int = 100
    maxStack: int = 1
    damage: int = 10
```

Created as asset files in the editor (ScriptableObject in Unity, .tres Resource in Godot). Lives in project assets.

---

### State Layer

What something IS right now. **Mutable.** One per logical occurrence.

```
class ItemState:
    dataId: string           // reference by ID, not direct
    durability: int
    quantity: int
    enchantments: list<string>

    cachedData: ItemData     // resolved at runtime from dataId

    constructor(data: ItemData, qty: int = 1):
        dataId = data.id
        cachedData = data
        durability = data.maxDurability
        quantity = qty
```

Why a separate class instead of storing values on the visual object? Because State must outlive the visual. When you pick up an item, the world object is destroyed but the State moves to inventory. When you drop it, a new visual is created from the same State.

Needs a parameterless constructor for deserialization — the loader creates the object first, then fills fields.

---

### Representation Layer

Visual/interactive form. **Disposable** — can be destroyed and recreated.

```
class WorldItem:
    state: ItemState

    setup(itemState):
        state = itemState
        mesh = state.cachedData.icon
        label = state.cachedData.displayName

    onInteract():
        if Inventory.tryAdd(state):
            destroySelf()    // visual dies, state lives in inventory
```

The core pattern in action:

- **Pick up:** destroy visual, state moves to inventory
- **Drop:** remove from inventory, create new visual with same state
- **Change scene:** visuals gone, state persists
- **Save/load:** save state, recreate visuals on load

---

## Database Pattern

Central lookup for finding data by string ID. Needed for loot tables, save/load resolution, console commands.

```
class ItemDatabase:
    items: map<string, ItemData>

    initialize():
        for each dataFile in assetsDirectory:
            item = load(dataFile)
            items[item.id] = item

    get(id) -> ItemData:
        return items[id]

    createInstance(id, qty = 1) -> ItemState:
        data = get(id)
        return new ItemState(data, qty) if data else null
```

**When you DON'T need this:** Few types, manually placed in editor — just reference data assets directly on the objects that use them.

---

## Saving and Loading

**Save State only.** Data already exists in project assets. Representation is recreated.

```
class SaveData:
    playerPosition: Vector3
    inventory: list<ItemState>
    quests: map<string, QuestState>
```

After loading, resolve each State's `dataId` against the database to reconnect Data references, then create Representations as needed.

What to save: current values, progress, world changes.
What NOT to save: Data templates (in assets already), visuals (recreated), computed values (recalculated).

---

## Extending Data Types

**Inheritance** — clear hierarchy, type-safe:
```
class WeaponData extends ItemData:
    attackDamage: int
    attackSpeed: float
```
Gets messy with deep hierarchies.

**Composition** — flexible, supports overlapping features:
```
class ItemData:
    id: string
    weapon: WeaponComponent      // null if not a weapon
    consumable: ConsumableComponent  // null if not consumable

class WeaponComponent:
    damage: int
    attackSpeed: float
```
More null checks, less type safety.

Simple game → inheritance. Complex game with many overlapping features → composition.

---

## Common Pitfalls

| Mistake | Fix |
|---|---|
| Modify Data at runtime | Mutate State, never Data — Data is a shared template |
| Store direct object reference in save | Use string ID, resolve from database on load |
| Logic in Data class | Keep logic in systems — Data defines, systems interpret |
| State only on the visual object | Separate State class — state must outlive visuals |
| Shallow copy of State with nested objects | Deep copy — nested lists/objects are shared by default |

---

## Key Takeaways

1. **Level 1** (extract values into data assets) — do this whenever you're duplicating prefabs to change numbers
2. **Level 2** (Data + State + Representation) — reach for this when state must outlive visuals, save/load, or data-without-objects
3. **Data** = template, read-only, shared
3. **State** = runtime values, mutable, per-instance
4. **Representation** = visual, disposable, recreated from State
5. **Data stores WHAT** (values, asset references) — **Representation is the THING** (structure, components)
6. **Reference by ID** for serialization
7. **Save State only** — Data is in assets, Representation is recreated

> [!quote] The Test (when using this pattern): Can someone create a new item type by ONLY creating a data asset file? If yes, you've achieved data-driven design.

---

See [[Layered Architecture]] for how to organize systems, managers, and components around this data.
