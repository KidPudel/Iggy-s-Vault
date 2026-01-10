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

## Three Layers

This is the foundational pattern.

|Layer|Data (Blueprint)|State (Runtime)|Representation (Node)|
|---|---|---|---|
|**What**|Template, the "platonic ideal"|Runtime state, "this specific one"|Visual/interactive form|
|**Class**|`ItemData`|`ItemState`|`WorldItem` (Node3D)|
|**Mutability**|Read-only|Mutable|Rebuilt on change|
|**Lifetime**|Shipped with game|Created during play|Created/destroyed as needed|
|**Location**|`res://` (serialized .tres)|Memory or `user://` (saves)|Scene tree|
|**Sharing**|One per type, shared by all|One per occurrence, unique|One per visible occurrence|
|**Created By**|`ResourceSaver.save()`|`.new()` constructor|`instantiate()`|

```
DATA (iron_axe.tres):                STATE (player's axe):           REPRESENTATION (world):
├─ id: "iron_axe"                    ├─ data: → iron_axe.tres        ├─ Mesh (axe model)
├─ display_name: "Iron Axe"          ├─ durability: 47  (current)    ├─ CollisionShape
├─ max_durability: 100  (maximum)    └─ enchantments: [fire, keen]   └─ state: → player's axe
├─ damage: 25
└─ icon: axe.png
```

**Data (Blueprint)** answers: "What CAN this be?" (max durability, base damage)  
**State (Runtime)** answers: "What IS this now?" (current durability, applied mods)  
**Representation (Node)** answers: "How does this appear?" (mesh, collision, interaction)

---

## Understanding OOP Instances vs Resources

This is critical to avoid confusion.

### Two Different Meanings of "Instance"

**OOP Instance** (created with `.new()`):

- Runtime object in memory
- Created by calling constructor: `ItemState.new()`
- Lives in [[RAM]], dies when reference count hits zero
- Can be serialized to disk as a Resource

**Serialized Resource** (created with `ResourceSaver.save()`):

- Data file on disk (`.tres`)
- Created in editor or by calling `ResourceSaver.save()`
- Lives in `res://` or `user://`
- Loaded into memory as OOP instance when needed

```gdscript
# OOP Instance - lives in RAM
var state := ItemState.new(iron_axe_data)

# Serialized Resource - lives on disk
ResourceSaver.save(state, "user://saves/player_axe.tres")

# Loading creates new OOP instance from serialized data
var loaded_state := ResourceLoader.load("user://saves/player_axe.tres")
```

### Why We Renamed ItemInstance → ItemState

The word "instance" is overloaded in programming:

- In OOP: any object created with `.new()`
- In Godot: serialized `.tres` files are called "resource instances"
- In game design: "an instance of an enemy" could mean the blueprint OR the runtime state

**ItemState** is clearer because:

- It describes what it holds: mutable runtime state
- It avoids confusion with OOP terminology
- It pairs clearly with ItemData (blueprint)

---

## When to Use What

Not everything needs all three layers.

### The Decision Tree

```
Does data need to outlive the node?
    │
    ├─ NO  → Data only (node holds runtime state directly)
    │        Enemy dies = data gone. That's fine.
    │        var health: int on the Enemy node.
    │
    └─ YES → Data + State
             Item picked up = node destroyed, state lives in inventory.
             
             Does it need visual representation in world?
                 │
                 ├─ YES → Data + State + Representation
                 │        Axe on ground, in inventory, equipped
                 │
                 └─ NO  → Data + State only
                          Quest progress, player stats
```

### Triggers for State Layer

|Question|If Yes → Need State|
|---|---|
|Persists after node destruction?|Inventory items outlive pickup nodes|
|Transfers between scenes?|Player stats cross levels|
|Gets saved to disk?|Save file needs current durability|
|Multiple nodes show same data?|Inventory slot + tooltip + equipped mesh|
|Variations of same blueprint?|Two axes with different durability|

### Quick Reference

|Situation|Pattern|
|---|---|
|Enemy dies and is gone|Data only, `var health` on node|
|Bullet, particle, temporary pickup|Data only|
|Inventory item|Data + State + Representation|
|Player stats|State only (or Data + State)|
|Quest progress|State only|
|Placeable furniture|Data + State + Representation|

---

## Implementation

### Data (Resource)

```gdscript
class_name ItemData
extends Resource

@export var id: StringName
@export var display_name: String
@export var icon: Texture2D
@export var max_durability: int = 100
@export var max_stack: int = 1
```

Create: `Right-click → New Resource → ItemData`  
Save as: `res://data/items/iron_axe.tres`

### State (Resource)

```gdscript
class_name ItemState
extends Resource

@export var data: ItemData
@export var durability: int = -1
@export var quantity: int = 1
@export var enchantments: Array[StringName] = []

func _init(item_data: ItemData = null, qty: int = 1) -> void:
    if item_data:
        data = item_data
        durability = item_data.max_durability
        quantity = qty
```

> [!warning] Constructor Must Have Defaults Godot needs parameterless construction for loading. Always use defaults.

### Representation (Node)

```gdscript
class_name WorldItem
extends Node3D

var state: ItemState

func setup(item_state: ItemState) -> void:
    state = item_state
    $Mesh.mesh = state.data.mesh
    $Label.text = state.data.display_name
```

### Database (for ID Lookup)

When you need lookup by string ID (loot tables, saves, console commands):

```gdscript
# item_database.gd (Autoload)
extends Node

var _items: Dictionary = {}  # id → ItemData

func _ready() -> void:
    var files := ResourceLoader.list_directory("res://data/items/")
    for file_name in files:
        if file_name.ends_with(".tres"):
            var item: ItemData = load("res://data/items/" + file_name)
            if item and item.id:
                _items[item.id] = item

func get_data(id: StringName) -> ItemData:
    return _items.get(id)

func create_state(id: StringName, qty: int = 1) -> ItemState:
    var item_data := get_data(id)
    return ItemState.new(item_data, qty) if item_data else null
```

> [!danger] Never Use DirAccess for res:// `DirAccess.open("res://")` breaks on export. Use `ResourceLoader.list_directory()`.

**When you DON'T need a database:** Few types, manually placed in editor → just use `@export var goblin_data: EnemyData`.

### Saving

```gdscript
class_name SaveGame
extends Resource

@export var player_position: Vector3
@export var inventory: Array[ItemState] = []
@export var quest_states: Dictionary = {}
```

```gdscript
# Save
ResourceSaver.save(save_data, "user://saves/slot_1.tres")

# Load — MUST use CACHE_MODE_IGNORE
var save := ResourceLoader.load(path, "", ResourceLoader.CACHE_MODE_IGNORE)
```

> [!danger] Always Use CACHE_MODE_IGNORE Without it, Godot returns cached resource. Mutations corrupt the cache. This is especially critical for save files where each load must create a fresh OOP instance.

---

## Common Patterns

### Spawning

```gdscript
func spawn_item(id: StringName, position: Vector3) -> void:
    var state := ItemDatabase.create_state(id)
    var node := preload("res://scenes/world_item.tscn").instantiate()
    node.setup(state)
    node.global_position = position
    world.add_child(node)
```

### Picking Up (State survives, node dies)

```gdscript
func interact() -> void:
    if Inventory.add(item_state):
        queue_free()  # Node dies, state lives in inventory
```

### Dropping (State gets new representation)

```gdscript
func drop(state: ItemState, position: Vector3) -> void:
    Inventory.remove(state)
    var node := preload("res://scenes/world_item.tscn").instantiate()
    node.setup(state)  # Same state, new node
    node.global_position = position
    world.add_child(node)
```

### Extending Types

```gdscript
class_name WeaponData
extends ItemData

@export var damage: int = 10
@export var attack_speed: float = 1.0
```

Or use composition:

```gdscript
class_name ItemData
extends Resource

@export var id: StringName
@export var weapon_data: WeaponData  # null if not a weapon
@export var consumable_data: ConsumableData  # null if not consumable
```

---

## Gotchas

|Smell|Problem|Fix|
|---|---|---|
|`item_data.damage = 999`|Mutates shared data|Mutate state, not data|
|`state.duplicate()`|Shallow copy, nested arrays shared|Use `duplicate(true)` or reconstruct|
|`resource_path` as ID|Breaks if you move files|Use explicit `id: StringName`|
|Missing `@export`|Field won't serialize|Add `@export` to all saved fields|
|Constructor with required params|Godot can't load it|Use defaults for all params|
|`DirAccess` for res://|Breaks on export|Use `ResourceLoader.list_directory()`|
|`ResourceLoader.load()` without cache mode|Returns cached, corrupted on mutation|Use `CACHE_MODE_IGNORE`|
|Calling data "blueprint"|Inconsistent terminology|Use Data/State terminology|

---

## Quick Reference

### Project Structure

```
res://
├── data/
│   ├── items/
│   │   ├── iron_axe.tres        (ItemData - blueprint)
│   │   └── health_potion.tres   (ItemData - blueprint)
│   ├── enemies/
│   │   └── goblin.tres          (EnemyData - blueprint)
│   └── quests/
│       └── main_quest.tres      (QuestData - blueprint)
├── scripts/
│   ├── data/
│   │   ├── item_data.gd
│   │   └── enemy_data.gd
│   ├── state/
│   │   ├── item_state.gd
│   │   └── quest_state.gd
│   └── databases/
│       └── item_database.gd
└── scenes/
    └── world_item.tscn          (WorldItem - representation)
```

### Storage Paths

|Path|Access|Use For|
|---|---|---|
|`res://`|Read-only|Data (blueprints), scenes, scripts|
|`user://`|Read-write|Saves (State), settings, logs|

### Checklist

**Data (Blueprints):**

- [ ] Extend `Resource`
- [ ] All fields `@export`
- [ ] Include `id: StringName`
- [ ] Saved as `.tres` in `res://data/`
- [ ] NEVER modified at runtime

**State (Runtime):**

- [ ] Extend `Resource`
- [ ] Hold reference to data (not copy)
- [ ] All saved fields `@export`
- [ ] Constructor has defaults
- [ ] Only this layer is mutable

**Representation (Node):**

- [ ] Holds reference to State
- [ ] Updates visuals when state changes
- [ ] Can be destroyed/recreated without losing state

**Database:**

- [ ] Use `ResourceLoader.list_directory()`
- [ ] NOT `DirAccess`

**Saving/Loading:**

- [ ] Use `CACHE_MODE_IGNORE` on load
- [ ] Only save State layer (data is already in res://)

---

## The Payoff

1. **Designers edit `.tres`, not code** — faster iteration
2. **Save system is trivial** — Resources serialize themselves
3. **New content = new data files** — no code changes
4. **Memory efficient** — thousands of states, few data blueprints
5. **Clear reasoning** — "what can it be?" vs "what is it now?" vs "how does it look?" never confused
6. **Flexible representation** — same state can appear in inventory UI, world, and equipment slots

> [!quote] The Ultimate Test Can someone create a new item type by ONLY creating a `.tres` file?

---

_For code organization (Systems, Managers, Nodes), see [[Layered Architecture in Godot]]._