# Data-Driven Design in Godot

> [!quote] The Core Insight Your game's items, enemies, quests — that IS the game. Code is just an interpreter.

---

## Prerequisites

This guide assumes you understand Godot Resources. If you're not familiar with how Resources work, serialization, or caching, read [[Godot Resource]] first.

---

## Mental Model: RAM vs Disk

Understanding the relationship between objects in memory and files on disk is crucial.

### What Actually Happens at Runtime

```gdscript
// At runtime, you work with OBJECTS IN RAM
var item_data = ItemData.new()  // Creates object in memory
item_data.damage = 25            // Modifies RAM object
```

**At this point:**

- `item_data` is a regular object in RAM (memory)
- It's just a class instance with fields
- It exists ONLY while the game is running
- When the game closes, it's gone forever

### What `.tres` Files Actually Are

```gdscript
// Saving converts RAM → Disk
ResourceSaver.save(item_data, "res://data/sword.tres")
```

**What just happened:**

1. Godot takes your RAM object
2. Serializes it (converts to text format)
3. Writes a `.tres` file to disk (hard drive)
4. That file persists after the game closes

**The `.tres` file is:**

- A text file on your hard drive
- A saved snapshot of the object's state
- Persistent storage (survives game restart)
- NOT an object you interact with directly

### Loading Brings Files Back to RAM

```gdscript
// Loading converts Disk → RAM
var loaded_data = load("res://data/sword.tres")
```

**What just happened:**

1. Godot reads the `.tres` file from disk
2. Deserializes it (converts text back to object)
3. Creates a new object in RAM
4. Returns that RAM object to you

**Now `loaded_data` is:**

- A regular object in RAM
- Identical to the original object that was saved
- What you actually interact with in code

### The Complete Flow

```
DEVELOPMENT TIME:                RUNTIME:
┌──────────────────┐            ┌─────────────────┐
│ Create in Editor │            │ Game starts     │
│ Save as .tres    │            │                 │
└────────┬─────────┘            └────────┬────────┘
         │                               │
         ▼                               ▼
┌──────────────────┐            ┌─────────────────┐
│ sword.tres       │ ─load()──> │ ItemData object │ <── You code with this
│ (on disk)        │            │ (in RAM)        │
│ Persistent ✓     │            │ Temporary       │
└──────────────────┘            └─────────────────┘
                                         │
                                         │ .new()
                                         ▼
                                ┌─────────────────┐
                                │ ItemState object│ <── You code with this
                                │ (in RAM)        │
                                │ Temporary       │
                                └────────┬────────┘
                                         │
                                         │ ResourceSaver.save()
                                         ▼
                                ┌─────────────────┐
                                │ save_slot1.tres │
                                │ (on disk)       │
                                │ Persistent ✓    │
                                └─────────────────┘
```

### Key Insights

**In your code, you ALWAYS work with objects in RAM:**

```gdscript
var data: ItemData = load("res://sword.tres")  // RAM object
var state: ItemState = ItemState.new(data)     // RAM object
state.durability = 50                           // Modifying RAM
```

**"Resource" just means:**

- The class inherits from `Resource`
- It can be serialized (converted to `.tres` format)
- You can save it with `ResourceSaver.save()`
- You can load it with `load()` or `preload()`

**It does NOT mean:**

- You're working with the file directly
- The object is magically stored on disk
- Changes automatically save to disk

### Practical Example

```gdscript
# Load data from disk → creates RAM object
var sword_data: ItemData = load("res://data/sword.tres")

# Create new state in RAM
var player_sword: ItemState = ItemState.new(sword_data)
player_sword.durability = 80  // Modifying RAM object

# At this point, nothing is saved to disk yet!

# Later, explicitly save to disk
var save_game = SaveGame.new()
save_game.inventory.append(player_sword)
ResourceSaver.save(save_game, "user://saves/slot1.tres")

# Now it's on disk and will persist
```

**Think of it like this:**

- `.tres` files are like save files or blueprints
- At runtime, you load them into RAM as objects
- You work with those RAM objects
- When you want to persist changes, you explicitly save back to disk

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

This is the foundational pattern for game entities that need persistence and visual representation.

### Conceptual Layers

| Layer          | Data                                   | State                                                               | Representation          |
| -------------- | -------------------------------------- | ------------------------------------------------------------------- | ----------------------- |
| **Purpose**    | Template, the "platonic ideal" of data | Runtime state, "this specific one", representing latest state saved | Visual/interactive form |
| **Class**      | `ItemData`                             | `ItemState`                                                         | `WorldItem` (Node3D)    |
| **Mutability** | Read-only                              | Mutable                                                             | Rebuilt on change       |
| **Answers**    | "What CAN this be?"                    | "What IS this now?"                                                 | "How does it appear?"   |

```
DATA (iron_axe.tres):                STATE (player's axe):           REPRESENTATION (world):
├─ id: "iron_axe"                    ├─ data: → iron_axe.tres        ├─ Mesh (axe model)
├─ display_name: "Iron Axe"          ├─ durability: 47  (current)    ├─ CollisionShape
├─ max_durability: 100  (maximum)    └─ enchantments: [fire, keen]   └─ state: → player's axe
├─ damage: 25
└─ icon: axe.png
```

### Where They Live at Runtime

Understanding the relationship between disk files and RAM objects:

| Layer              | In RAM (Runtime)                                    | On Disk (Persistent)                                   |
| ------------------ | --------------------------------------------------- | ------------------------------------------------------ |
| **Data**           | `ItemData` object (loaded, cached, read-only)       | `res://data/items/iron_axe.tres` (created from editor) |
| **State**          | `ItemState` object (created with `.new()`, mutable) | `user://saves/slot1.tres` (when saved)                 |
| **Representation** | `WorldItem` node (in scene tree)                    | Not saved (recreated from State)                       |

**Key distinction:**

- **Data layer** = `.tres` file in `res://` → loaded once → cached RAM object (shared, read-only)
- **State layer** = RAM object created at runtime → optionally saved to `user://` as `.tres` with `ResourceSaver.save()`
- **Representation layer** = Node in scene tree → never saved (always recreated)

**Example flow:**

```gdscript
# Load data from disk → creates cached RAM object
var iron_axe_data: ItemData = load("res://data/items/iron_axe.tres")

# Create state in RAM (not saved yet)
var player_axe: ItemState = ItemState.new(iron_axe_data)
player_axe.durability = 47  # Modify RAM object

# Create representation in scene tree
var world_item: WorldItem = preload("res://scenes/world_item.tscn").instantiate()
world_item.setup(player_axe)  # Links to state
add_child(world_item)

# Later: explicitly save state to disk
ResourceSaver.save(player_axe, "user://saves/player_items/axe_001.tres")
```

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
|Variations of same data?|Two axes with different durability|

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

### Data Layer (Resource)

```gdscript
class_name ItemData
extends Resource

@export var id: StringName
@export var display_name: String
@export var icon: Texture2D
@export var max_durability: int = 100
@export var max_stack: int = 1
@export var damage: int = 10
```

Create: `Right-click → New Resource → ItemData`  
Save as: `res://data/items/iron_axe.tres`

**Key characteristics:**

- Defines what something CAN be
- Immutable at runtime
- Shared by all instances
- Stored in `res://`

### State Layer (Resource)

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

**Key characteristics:**

- Holds current/mutable values
- References Data (doesn't copy it)
- One per logical occurrence
- Saved to `user://` (if persistent)

### Representation Layer (Node)

```gdscript
class_name WorldItem
extends Node3D

var state: ItemState

func setup(item_state: ItemState) -> void:
    state = item_state
    $Mesh.mesh = state.data.mesh
    $Label.text = state.data.display_name
    
func _on_interact() -> void:
    if Inventory.add(state):
        queue_free()  # Node dies, state lives
```

**Key characteristics:**

- Visual/interactive form
- Displays State data
- Can be destroyed/recreated without losing state
- Lives in scene tree

---

## Database Pattern

Use when you need to look up data by string ID (loot tables, saves, console commands).

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

**When you DON'T need a database:**  
Few types, manually placed in editor → just use `@export var goblin_data: EnemyData`.

---

## Saving and Loading

### Save Structure

```gdscript
class_name SaveGame
extends Resource

@export var player_position: Vector3
@export var inventory: Array[ItemState] = []
@export var quest_states: Dictionary = {}
```

### Save/Load Operations

```gdscript
# Saving is straightforward
ResourceSaver.save(save_data, "user://saves/slot_1.tres")

# Loading MUST use CACHE_MODE_IGNORE
var save := ResourceLoader.load(
    "user://saves/slot_1.tres",
    "",
    ResourceLoader.CACHE_MODE_IGNORE
)
```

> [!danger] Always Use CACHE_MODE_IGNORE for Save Files Without it, Godot returns the cached Resource. Mutations corrupt the cache, breaking subsequent loads. See [[Resources in Godot]] for details.

**What to save:**

- State layer (current durability, enchantments, quantity)
- Player progress (position, quest completion)
- World changes (opened chests, defeated bosses)

**What NOT to save:**

- Data layer (already in `res://`)
- Nodes/scenes (recreate from state on load)
- Computed values (recalculate from saved data)

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

### Transferring Between Scenes

```gdscript
# Scene A: Store state before changing scenes
Global.player_state = player.get_state()
get_tree().change_scene_to_file("res://scenes/level_2.tscn")

# Scene B: Restore state after loading
func _ready() -> void:
    if Global.player_state:
        player.restore_state(Global.player_state)
```

---

## Extending Data Types

### Inheritance Approach

```gdscript
class_name WeaponData
extends ItemData

@export var damage: int = 10
@export var attack_speed: float = 1.0
@export var weapon_type: WeaponType
```

**Pros:** Type-safe, clear hierarchy  
**Cons:** Deep hierarchies get messy

### Composition Approach

```gdscript
class_name ItemData
extends Resource

@export var id: StringName
@export var display_name: String
@export var weapon_data: WeaponData  # null if not a weapon
@export var consumable_data: ConsumableData  # null if not consumable
@export var armor_data: ArmorData  # null if not armor
```

**Pros:** Flexible, supports multiple "aspects"  
**Cons:** More null checks, less type safety

Choose based on your game's needs. Simple games → inheritance. Complex games with many overlapping features → composition.

---

## Architecture Guidelines

### Ownership Flows Downward

```gdscript
# ✅ Good: Parent owns children
class_name Inventory
extends Resource
@export var items: Array[ItemState] = []

# ❌ Bad: Child points to parent
class_name ItemState
extends Resource
@export var inventory: Inventory  # Circular reference!
```

Store "owns many" relationships on the parent, not "belongs to" on children. This prevents circular references and simplifies serialization.

### Separate Static from Mutable

|Path|Mutability|Use For|
|---|---|---|
|`res://`|Read-only|Data (templates, definitions)|
|`user://`|Read-write|State (saves, settings, progress)|

Never modify Resources loaded from `res://` at runtime. If you need mutable versions, create State instances that reference the Data.

### Keep Logic in Systems, Data in Resources

```gdscript
# ❌ Bad: Logic in data
class_name ItemData
extends Resource
func use(player: Player) -> void:
    player.heal(healing_amount)

# ✅ Good: Logic in system
class_name ItemSystem
func use_item(item_state: ItemState, player: Player) -> void:
    if item_state.data.consumable_data:
        player.heal(item_state.data.consumable_data.healing_amount)
```

Resources define data. Systems interpret data and apply logic.

---

## Common Pitfalls

|Smell|Problem|Fix|
|---|---|---|
|`item_data.damage = 999`|Mutates shared data|Mutate state, not data|
|`state.duplicate()`|Shallow copy, nested arrays shared|Use `duplicate(true)`|
|`resource_path` as ID|Breaks if you move files|Use explicit `id: StringName`|
|Missing `@export`|Field won't serialize|Add `@export` to all saved fields|
|Logic in Resources|Hard to test, couples data to behavior|Move logic to Systems|
|`DirAccess` for res://|Breaks on export|Use `ResourceLoader.list_directory()`|
|Loading without `CACHE_MODE_IGNORE`|Corrupted save data|Always use for mutable files|

---

## Project Structure

```
res://
├── data/                           # All data definitions
│   ├── items/
│   │   ├── iron_axe.tres          (ItemData)
│   │   ├── steel_sword.tres       (WeaponData)
│   │   └── health_potion.tres     (ItemData)
│   ├── enemies/
│   │   └── goblin.tres            (EnemyData)
│   └── quests/
│       └── main_quest.tres        (QuestData)
├── scripts/
│   ├── data/                      # Data layer classes
│   │   ├── item_data.gd
│   │   ├── weapon_data.gd
│   │   └── enemy_data.gd
│   ├── state/                     # State layer classes
│   │   ├── item_state.gd
│   │   └── quest_state.gd
│   ├── systems/                   # Game logic
│   │   ├── inventory_system.gd
│   │   └── combat_system.gd
│   └── databases/                 # Autoloads
│       ├── item_database.gd
│       └── quest_database.gd
└── scenes/
    ├── world_item.tscn            (WorldItem - representation)
    └── enemy.tscn                 (Enemy - representation)
```

---

## Checklist

### Data Layer

- [ ] Extend `Resource`
- [ ] All fields `@export`
- [ ] Include `id: StringName`
- [ ] Saved as `.tres` in `res://data/`
- [ ] NEVER modified at runtime
- [ ] Defines what something CAN be

### State Layer

- [ ] Extend `Resource`
- [ ] References Data (doesn't copy it)
- [ ] All saved fields `@export`
- [ ] Constructor has defaults
- [ ] Only this layer is mutable
- [ ] Represents what something IS now

### Representation Layer

- [ ] Holds reference to State
- [ ] Updates visuals when state changes
- [ ] Can be destroyed/recreated without losing state
- [ ] Handles user interaction

### Database

- [ ] Use `ResourceLoader.list_directory()`
- [ ] NOT `DirAccess`
- [ ] Indexes by `id: StringName`

### Saving/Loading

- [ ] Use `CACHE_MODE_IGNORE` on load
- [ ] Only save State layer
- [ ] Data layer stays in `res://`

---

## The Payoff

1. **Designers edit `.tres`, not code** — faster iteration, no programming needed
2. **Save system is trivial** — Resources serialize themselves automatically
3. **New content = new data files** — no code changes required
4. **Memory efficient** — thousands of states, few data templates
5. **Clear separation** — "what can it be?" vs "what is it now?" vs "how does it look?"
6. **Flexible representation** — same state can appear in inventory UI, world, and equipment slots
7. **Easy to test** — data is isolated from logic

> [!quote] The Ultimate Test Can someone create a new item type by ONLY creating a `.tres` file?

If yes, you've achieved true data-driven design.

---

## Further Reading

- [[Resources in Godot]] — Foundation for understanding serialization
- [[Layered Architecture in Godot]] — How to organize systems and managers

---

**Remember:** Data is the game. Code just interprets it.