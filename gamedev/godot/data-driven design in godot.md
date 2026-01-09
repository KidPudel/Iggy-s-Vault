# Data-Driven Design in Godot

> [!quote] The Core Insight Your game's items, enemies, quests — that IS the game. Code is just an interpreter.

---

## The Problem

Without data-driven design:

```
"I need to code an iron axe"
→ IronAxe.gd with hardcoded stats
→ Copy-paste for SteelAxe.gd, MithrilAxe.gd...
→ Designer wants to tweak damage
→ Edit 47 files
```

With data-driven design:

```
"What data describes an axe?"
→ Define shape (Resource script)
→ Create iron_axe.tres, steel_axe.tres...
→ One script interprets any axe
→ Designer tweaks in inspector, zero code changes
```

**The shift:** Stop thinking "code X." Start thinking "what data describes X?"

---

## Definition vs Instance

This is the foundational pattern.

| Aspect         | Definition                  | Instance                              |
| -------------- | --------------------------- | ------------------------------------- |
| **What**       | Template, the truth         | Runtime state, "this specific one"    |
| **Mutability** | Read-only                   | Mutable                               |
| **Lifetime**   | Shipped with game           | Created during play                   |
| **Location**   | `res://`                    | Created in memory, saved to `user://` |
| **Sharing**    | One per type, shared by all | One per occurrence, unique            |

```
DEFINITION (iron_axe.tres):          INSTANCE (player's axe):
├─ id: "iron_axe"                    ├─ definition: → iron_axe.tres
├─ display_name: "Iron Axe"          ├─ durability: 47  (current)
├─ max_durability: 100  (maximum)    └─ enchantments: [fire, keen]
├─ damage: 25
└─ icon: axe.png
```

**Definition** answers: "What CAN this be?" (max durability, base damage) **Instance** answers: "What IS this now?" (current durability, applied mods)

---

## When to Use What

Not everything needs both layers.

### The Decision

```
Does data need to survive node destruction?
    │
    ├─ NO  → Definition only (node holds runtime state)
    │        Enemy dies = data gone. That's fine.
    │
    └─ YES → Definition + Instance
             Item picked up = node gone, data lives in inventory.
```

### Triggers for Instance

|Question|If Yes → Need Instance|
|---|---|
|Persists after node destruction?|Inventory items outlive pickup nodes|
|Transfers between scenes?|Player stats cross levels|
|Gets saved to disk?|Save file needs current durability|
|Multiple nodes show same data?|Inventory slot + tooltip|
|Variations of same definition?|Two axes with different durability|

### Quick Reference

|Situation|Pattern|
|---|---|
|Enemy dies and is gone|Definition only, `var health` on node|
|Bullet, particle, temporary pickup|Definition only|
|Inventory item|Definition + Instance|
|Player stats|Definition + Instance (or just Instance)|
|Quest progress|Instance|
|Placeable furniture|Definition + Instance|

---

## Implementation

### Definition (Resource)

```gdscript
class_name ItemDefinition
extends Resource

@export var id: StringName
@export var display_name: String
@export var icon: Texture2D
@export var max_durability: int = 100
@export var max_stack: int = 1
```

Create: `Right-click → New Resource → ItemDefinition` Save as: `res://data/items/iron_axe.tres`

### Instance (Resource)

```gdscript
class_name ItemInstance
extends Resource

@export var definition: ItemDefinition
@export var durability: int = -1
@export var quantity: int = 1
@export var enchantments: Array[StringName] = []

func _init(def: ItemDefinition = null, qty: int = 1) -> void:
    if def:
        definition = def
        durability = def.max_durability
        quantity = qty
```

> [!warning] Constructor Must Have Defaults Godot needs parameterless construction for loading. Always use defaults.

### Database (for ID Lookup)

When you need lookup by string ID (loot tables, saves, console commands):

```gdscript
# item_database.gd (Autoload)
extends Node

var _items: Dictionary = {}  # id → ItemDefinition

func _ready() -> void:
    var files := ResourceLoader.list_directory("res://data/items/")
    for file_name in files:
        if file_name.ends_with(".tres"):
            var item: ItemDefinition = load("res://data/items/" + file_name)
            if item and item.id:
                _items[item.id] = item

func get_definition(id: StringName) -> ItemDefinition:
    return _items.get(id)

func create_instance(id: StringName, qty: int = 1) -> ItemInstance:
    var def := get_definition(id)
    return ItemInstance.new(def, qty) if def else null
```

> [!danger] Never Use DirAccess for res:// `DirAccess.open("res://")` breaks on export. Use `ResourceLoader.list_directory()`.

**When you DON'T need a database:** Few types, manually placed in editor → just use `@export var goblin: EnemyDefinition`.

### Saving

```gdscript
class_name SaveGame
extends Resource

@export var player_position: Vector3
@export var inventory: Array[ItemInstance] = []
@export var quest_states: Dictionary = {}
```

```gdscript
# Save
ResourceSaver.save(save_data, "user://saves/slot_1.tres")

# Load — MUST use CACHE_MODE_IGNORE
var save := ResourceLoader.load(path, "", ResourceLoader.CACHE_MODE_IGNORE)
```

> [!danger] Always Use CACHE_MODE_IGNORE Without it, Godot returns cached resource. Mutations corrupt the cache.

---

## Common Patterns

### Spawning

```gdscript
func spawn_item(id: StringName, position: Vector3) -> void:
    var instance := ItemDatabase.create_instance(id)
    var node := preload("res://scenes/world_item.tscn").instantiate()
    node.setup(instance)
    node.global_position = position
    world.add_child(node)
```

### Picking Up (Instance survives node)

```gdscript
func interact() -> void:
    if Inventory.add(item_instance):
        queue_free()  # Node dies, instance lives
```

### Dropping (Instance gets new node)

```gdscript
func drop(instance: ItemInstance, position: Vector3) -> void:
    Inventory.remove(instance)
    var node := preload("res://scenes/world_item.tscn").instantiate()
    node.setup(instance)  # Same instance, new node
    node.global_position = position
    world.add_child(node)
```

### Extending Types

```gdscript
class_name WeaponDefinition
extends ItemDefinition

@export var damage: int = 10
@export var attack_speed: float = 1.0
```

Or use composition:

```gdscript
class_name ItemDefinition
extends Resource

@export var id: StringName
@export var weapon_data: WeaponData  # null if not a weapon
@export var consumable_data: ConsumableData  # null if not consumable
```

---

## Gotchas

|Smell|Problem|Fix|
|---|---|---|
|`definition.damage = 999`|Mutates shared template|Mutate instance, not definition|
|`instance.duplicate()`|Shallow copy, nested arrays shared|Use `duplicate(true)` or reconstruct|
|`resource_path` as ID|Breaks if you move files|Use explicit `id: StringName`|
|Missing `@export`|Field won't serialize|Add `@export` to all saved fields|
|Constructor with required params|Godot can't load it|Use defaults for all params|
|`DirAccess` for res://|Breaks on export|Use `ResourceLoader.list_directory()`|
|`ResourceLoader.load()` without cache mode|Returns cached, corrupted on mutation|Use `CACHE_MODE_IGNORE`|

---

## Quick Reference

### Project Structure

```
res://
├── data/
│   ├── items/
│   │   ├── iron_axe.tres
│   │   └── health_potion.tres
│   ├── enemies/
│   │   └── goblin.tres
│   └── quests/
│       └── main_quest.tres
├── scripts/
│   ├── definitions/
│   │   ├── item_definition.gd
│   │   └── enemy_definition.gd
│   ├── instances/
│   │   └── item_instance.gd
│   └── databases/
│       └── item_database.gd
└── scenes/
    └── world_item.tscn
```

### Storage Paths

|Path|Access|Use For|
|---|---|---|
|`res://`|Read-only|Definitions, scenes, scripts|
|`user://`|Read-write|Saves, settings, logs|

### Checklist

**Definitions:**

- [ ] Extend `Resource`
- [ ] All fields `@export`
- [ ] Include `id: StringName`
- [ ] Saved as `.tres` in `res://data/`
- [ ] NEVER modified at runtime

**Instances:**

- [ ] Extend `Resource`
- [ ] Hold reference to definition (not copy)
- [ ] All saved fields `@export`
- [ ] Constructor has defaults

**Database:**

- [ ] Use `ResourceLoader.list_directory()`
- [ ] NOT `DirAccess`

**Saving/Loading:**

- [ ] Use `CACHE_MODE_IGNORE` on load

---

## The Payoff

1. **Designers edit `.tres`, not code** — faster iteration
2. **Save system is trivial** — Resources serialize themselves
3. **New content = new data files** — no code changes
4. **Memory efficient** — thousands of instances, few definitions
5. **Clear reasoning** — "what can it be?" vs "what is it now?" never confused

> [!quote] The Ultimate Test Can someone create a new item type by ONLY creating a `.tres` file?

---

_For code organization (Systems, Managers, Nodes), see [[Layered Architecture in Godot]]._