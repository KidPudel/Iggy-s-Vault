Case: You are picking up an axe from the world (some Node3D, with area and script on picking up logic), obviously we don't want to collect the same object, but rather a **data representation** of the object, that doesn't care about anything, except for data it has itself.
This is a perfect case for [[data-driven design]] to be introduced in Godot.

In Godot to do so, we can simply create a data model, which could be a literal data structure, that acts as a database stored in some basic script/class
```gdscript
class_name Items

const Database = {
	"pickaxe": {
		"name": "Pickaxe"
		"scene": "res://path/to/pickaxe.glb"
	},
	"sword": {
		"name": "Sword"
		"scene": "res://path/to/sword.glb"
	}
}
```

But this approach has several crucial problems:
1. We can easily create a typo in node that uses this like type"swort", which will be a runtime error
2. We can simply move scenes to other places, and the same error will happen
3. Maintaining 100 objects is still hard taking into account 1st and 2nd problems.

We can utilize godot specific feature called [[Godot resource]], which is a data class/structure, which allows for modeling a [[static]] data.
```gdscript
class_name Item extends Resource

@export var name: String
@export var scene: PackedScene # godot will automatically fix the path if we move it somewhere
```

Which then allows as to create specific resources based on our new `Item` type, like `Pickaxe`, `Sword`, etc. and now we can securely reference concrete setup resources.

**If your CardResource instances exist only inside a ColumnResource and nowhere else, there’s no need to save them individually as separate .tres or .tscn files.**


# Data-Driven Design in Godot

> [!quote] The Core Insight Your game's items, enemies, quests, dialogue — that **IS** the game. The code is just an interpreter.

---

## The Mental Shift

Most developers start here:

```
"I need to code an iron axe"
→ Write IronAxe.gd with hardcoded stats
→ Copy-paste for SteelAxe.gd
→ Copy-paste for MithrilAxe.gd
→ Designer wants to tweak damage
→ Edit 47 files
→ Regret life choices
```

Data-driven developers start here:

```
"What IS an axe?"
→ Define the shape of axe data (Resource)
→ Create iron_axe.tres, steel_axe.tres, mithril_axe.tres
→ One script interprets any axe
→ Designer tweaks damage in inspector
→ Zero code changes
→ Peace
```

**The shift:** Stop thinking "I need to code X." Start thinking "What data describes X, and what code interprets that data?"

---

## The Fuel and Engine Metaphor

```
DATA (Fuel)                         CODE (Engine)
─────────────────────────────────   ─────────────────────────────────
The creative artifact               The infrastructure
What makes YOUR game unique         What makes games work generally
Changes constantly during dev       Changes rarely once working
Designed by designers               Written by programmers
Lives in .tres files                Lives in .gd files

iron_axe.tres:                      item_system.gd:
  damage: 25                          func use(item):
  weight: 3.5                           # interprets ANY item
  icon: axe.png                         # doesn't know axes exist
```

Same principle everywhere:

- HTML is the document, browser is the renderer
- SQL data is the content, database engine is the interpreter
- Game data is the creative work, game code is the runtime

**You don't embed the document inside the browser's source code.** Why embed your game content inside your game code?

---

## The Two Types of Data

This is the foundational pattern. Everything else builds on it.

### Definition: What Something IS

```
┌─────────────────────────────────────────────────────────────┐
│                      DEFINITION                             │
│                                                             │
│   "The Platonic ideal of an Iron Axe"                       │
│                                                             │
│   - Static template                                         │
│   - Read-only at runtime                                    │
│   - Shared across ALL iron axes in the game                 │
│   - Lives in res:// (shipped with game)                     │
│   - Authored in editor, never touched by code               │
│                                                             │
│   Contains:                                                 │
│   - id: "iron_axe"                                          │
│   - display_name: "Iron Axe"                                │
│   - max_durability: 100        ← MAXIMUM, not current       │
│   - damage: 25                                              │
│   - icon: preloaded texture                                 │
└─────────────────────────────────────────────────────────────┘
```

### Instance: What's Happening NOW

```
┌─────────────────────────────────────────────────────────────┐
│                       INSTANCE                              │
│                                                             │
│   "This specific axe in the player's inventory"             │
│                                                             │
│   - Runtime state                                           │
│   - Mutable                                                 │
│   - Unique per occurrence                                   │
│   - Created during gameplay                                 │
│   - Saved to user:// (player's save file)                   │
│                                                             │
│   Contains:                                                 │
│   - definition: → iron_axe.tres    ← REFERENCE, not copy    │
│   - durability: 47                 ← CURRENT, not maximum   │
│   - enchantments: [fire, keen]     ← THIS axe's mods        │
└─────────────────────────────────────────────────────────────┘
```

### Why This Split Matters

|Problem|Without Split|With Split|
|---|---|---|
|100 iron axes in world|100 copies of "damage: 25, name: Iron Axe, ..."|100 refs to 1 definition|
|Designer tweaks damage|Edit code or 100 data files|Edit 1 .tres file|
|Save game|Store everything about every item|Store only what changed|
|"What can an item be?"|Grep through codebase|Read ItemDefinition.gd|
|Memory usage|Duplicated strings, textures|Shared references|

> [!important] The Key Insight Definition answers "what CAN this be?" (maximum durability, base damage) Instance answers "what IS this right now?" (current durability, applied enchantments)

---

## The Decision Framework

Not everything needs both layers. Here's how to decide:

```
                    Does data need to survive node destruction?
                                      │
                         ┌────────────┴────────────┐
                         │                         │
                        NO                        YES
                         │                         │
                         ▼                         ▼
              ┌─────────────────────┐   ┌─────────────────────┐
              │  DEFINITION ONLY    │   │ DEFINITION+INSTANCE │
              │                     │   │                     │
              │  Node holds state   │   │  Separate Resource  │
              │  var health: int    │   │  for runtime state  │
              │                     │   │                     │
              │  Examples:          │   │  Examples:          │
              │  - Enemy (dies=gone)│   │  - Inventory item   │
              │  - Bullet           │   │  - Player stats     │
              │  - Particle         │   │  - Quest progress   │
              │  - Temporary pickup │   │  - Persistent NPC   │
              └─────────────────────┘   └─────────────────────┘
```

### When Definition-Only is Enough

```gdscript
# Enemy dies, data dies with it. No save, no transfer.
extends CharacterBody3D

@export var definition: EnemyDefinition  # Template
var health: int                          # Runtime state ON THE NODE

func _ready() -> void:
    health = definition.max_health

func die() -> void:
    queue_free()  # Node gone, data gone. That's fine.
```

### When You Need Instance

The moment you answer "yes" to any of these:

|Question|If Yes → Need Instance|
|---|---|
|Does it persist after node destruction?|Inventory items outlive pickup nodes|
|Does it transfer between scenes?|Player stats cross level boundaries|
|Does it get saved to disk?|Save file needs durability, enchantments|
|Do multiple nodes display the same data?|Inventory slot + tooltip show same item|
|Can there be variations of the same definition?|Two iron axes with different durability|

---

## Implementation in Godot

### Definition (Resource)

```gdscript
# item_definition.gd
class_name ItemDefinition
extends Resource

@export var id: StringName           # Unique identifier for lookup
@export var display_name: String     # What player sees
@export var description: String
@export var icon: Texture2D
@export var max_durability: int = 100
@export var max_stack: int = 1
@export var weight: float = 1.0
@export var value: int = 0
```

Create in editor: `Right-click → New Resource → ItemDefinition` Save as: `res://data/items/iron_axe.tres`

> [!tip] The id Field Always include a `StringName` id. You'll need it for:
> 
> - Save files (store id, not resource path)
> - Network sync (send id, not entire definition)
> - Console commands ("give iron_axe")
> - Database lookup

### Instance (Resource)

```gdscript
# item_instance.gd
class_name ItemInstance
extends Resource

@export var definition: ItemDefinition  # Reference to template
@export var durability: int = -1        # -1 = use definition.max_durability
@export var quantity: int = 1
@export var enchantments: Array[StringName] = []
@export var custom_data: Dictionary = {}

# Godot requires parameterless constructor for Resource instantiation
func _init(def: ItemDefinition = null, qty: int = 1) -> void:
    if def:
        definition = def
        durability = def.max_durability
        quantity = qty

# Convenience accessors — Instance speaks for itself
var display_name: String:
    get: return definition.display_name if definition else ""

var icon: Texture2D:
    get: return definition.icon if definition else null

var is_broken: bool:
    get: return durability <= 0

func damage(amount: int) -> bool:
    durability = max(0, durability - amount)
    return is_broken
```

> [!warning] @export is Required for Saving Only `@export` properties are serialized by `ResourceSaver`. If you forget `@export`, that data won't save.

### Creating Instances

```gdscript
# From code
var axe := ItemInstance.new(iron_axe_definition, 1)

# From database (preferred)
var axe := ItemDatabase.create_instance(&"iron_axe")
```

---

## The Database Pattern

When you have many definitions and need lookup by ID.

### When You Need a Database

|Situation|Need Database?|
|---|---|
|3 enemy types, manually placed in editor|No — use @export|
|50 item types, spawned from loot tables|Yes|
|Procedural generation from string IDs|Yes|
|Save file stores item IDs|Yes|
|Console commands like "give iron_axe"|Yes|
|Network sends item IDs|Yes|

### When You Don't

```gdscript
# Direct references — no database needed
# You know exactly what you need at edit time
@export var goblin: EnemyDefinition
@export var skeleton: EnemyDefinition

func spawn_wave_1() -> void:
    spawn(goblin)
    spawn(skeleton)
```

### Implementation

```gdscript
# item_database.gd (Autoload)
class_name ItemDatabase
extends Node

var _items: Dictionary = {}  # id → ItemDefinition

func _ready() -> void:
    _load_all_items()

func _load_all_items() -> void:
    # ResourceLoader.list_directory works in editor AND export
    var files := ResourceLoader.list_directory("res://data/items/")
    for file_name in files:
        if file_name.ends_with(".tres"):
            var item: ItemDefinition = load("res://data/items/" + file_name)
            if item and item.id:
                _items[item.id] = item

func get_definition(id: StringName) -> ItemDefinition:
    return _items.get(id)

func create_instance(id: StringName, quantity: int = 1) -> ItemInstance:
    var def := get_definition(id)
    if not def:
        push_error("Unknown item: %s" % id)
        return null
    return ItemInstance.new(def, quantity)

func get_all_ids() -> Array[StringName]:
    var ids: Array[StringName] = []
    ids.assign(_items.keys())
    return ids
```

> [!danger] Never Use DirAccess for res:// Scanning `DirAccess.open("res://...")` works in editor but **breaks on export**. Files become `.import` remaps and `DirAccess` doesn't resolve them.
> 
> Always use `ResourceLoader.list_directory()` instead.

---

## Saving and Loading

### The Magic of Resource Serialization

When Godot saves your ItemInstance, it stores Definition as a resource path:

```
[resource]
script = ExtResource("res://scripts/item_instance.gd")
definition = ExtResource("res://data/items/iron_axe.tres")
durability = 47
quantity = 1
enchantments = ["fire", "keen"]
```

On load, Godot resolves the path automatically. Your instances reconnect to shared definitions without any extra work.

### SaveGame Resource

Wrap all saveable state in one resource:

```gdscript
# save_game.gd
class_name SaveGame
extends Resource

@export var version: int = 1
@export var player_position: Vector3
@export var player_health: int
@export var inventory_items: Array[ItemInstance] = []
@export var chest_contents: Dictionary = {}  # chest_id → Array[ItemInstance]
@export var quest_states: Dictionary = {}
@export var play_time_seconds: float = 0.0
@export var timestamp: String = ""
```

### Save Manager

```gdscript
# save_manager.gd (Autoload)
extends Node

const SAVE_DIR := "user://saves/"
const SAVE_EXT := ".tres"

func save_game(slot: int) -> bool:
    _ensure_save_dir()
    
    var save := SaveGame.new()
    save.version = 1
    save.timestamp = Time.get_datetime_string_from_system()
    save.player_position = player.global_position
    save.player_health = player.health
    save.inventory_items = GameSystems.inventory.get_all_items()
    save.quest_states = GameSystems.quests.get_all_states()
    
    var path := _get_save_path(slot)
    var error := ResourceSaver.save(save, path)
    
    if error != OK:
        push_error("Failed to save: %s" % error_string(error))
        return false
    return true

func load_game(slot: int) -> bool:
    var path := _get_save_path(slot)
    
    if not ResourceLoader.exists(path):
        push_warning("No save file: %s" % path)
        return false
    
    # CRITICAL: CACHE_MODE_IGNORE prevents corrupted state
    var save: SaveGame = ResourceLoader.load(
        path, "", ResourceLoader.CACHE_MODE_IGNORE
    )
    
    player.global_position = save.player_position
    player.health = save.player_health
    GameSystems.inventory.set_all_items(save.inventory_items)
    GameSystems.quests.set_all_states(save.quest_states)
    
    return true

func _get_save_path(slot: int) -> String:
    return SAVE_DIR + "save_%d%s" % [slot, SAVE_EXT]

func _ensure_save_dir() -> void:
    DirAccess.make_dir_recursive_absolute(SAVE_DIR)
```

> [!danger] Always Use CACHE_MODE_IGNORE on Load Without it, Godot returns the cached resource. Mutating it corrupts the cache. Next load? Corrupted data.
> 
> ```gdscript
> # BAD — returns cached resource, mutations persist
> var save = ResourceLoader.load(path)
> 
> # GOOD — fresh copy every time
> var save = ResourceLoader.load(path, "", ResourceLoader.CACHE_MODE_IGNORE)
> ```

---

## Storage Paths

### res:// — Project Resources (Read-Only)

- Your project folder in editor
- Packed into `.pck` on export
- **Player cannot write here**
- Use for: definitions, scenes, scripts, static assets

### user:// — User Data (Read-Write)

- Per-user writable folder
- Platform-specific locations:
    - Windows: `%APPDATA%/Godot/app_userdata/YourGame/`
    - macOS: `~/Library/Application Support/Godot/app_userdata/YourGame/`
    - Linux: `~/.local/share/godot/app_userdata/YourGame/`
- Use for: saves, settings, logs, screenshots

---

## Design Process: Data First

Before writing any code, answer these questions:

### 1. What Data Exists?

List every piece of information for the feature:

```
ITEM:
- identifier (for lookup, saves)
- display name, description
- icon texture
- max durability, weight
- stackable? max stack size
- equip slot (if equipment)
- use effect (if consumable)
```

### 2. What's Static vs Dynamic?

|Static (Definition)|Dynamic (Instance)|
|---|---|
|id, display_name|—|
|max_durability|current durability|
|max_stack|current quantity|
|base_damage|enchantment bonuses|
|icon|—|
|weight|—|

### 3. What's the Lifecycle?

```
ITEM LIFECYCLE:
1. Definition exists in res://data/items/
2. Instance created when: spawned in world, given by NPC, crafted
3. Instance modified when: used, damaged, enchanted
4. Instance destroyed when: consumed, broken, dropped in lava
5. Instance persisted when: game saved
6. Instance restored when: game loaded
```

### 4. What Operations Happen?

These become your system/manager methods:

```
ITEM OPERATIONS:
- create(definition_id) → Instance
- add_to_inventory(instance)
- remove_from_inventory(instance)
- use(instance)
- damage(instance, amount)
- repair(instance, amount)
- split_stack(instance, amount) → new Instance
- merge_stacks(instance_a, instance_b)
- drop(instance, position)
- save() / load()
```

---

## Extending the Pattern

### Inheritance for Specialized Types

```gdscript
# weapon_definition.gd
class_name WeaponDefinition
extends ItemDefinition

@export var damage: int = 10
@export var attack_speed: float = 1.0
@export var damage_type: StringName = &"physical"
@export var range: float = 1.5
```

```gdscript
# consumable_definition.gd
class_name ConsumableDefinition
extends ItemDefinition

@export var heal_amount: int = 0
@export var mana_restore: int = 0
@export var buff: BuffDefinition
@export var buff_duration: float = 0.0
```

### Instance-Specific State

```gdscript
# weapon_instance.gd
class_name WeaponInstance
extends ItemInstance

@export var kills: int = 0
@export var enchantments: Array[EnchantmentInstance] = []

var total_damage: int:
    get:
        var base: int = (definition as WeaponDefinition).damage
        var bonus: int = 0
        for ench in enchantments:
            bonus += ench.damage_bonus
        return base + bonus
```

### Composition over Inheritance

Sometimes inheritance isn't the right fit. Use composition:

```gdscript
# item_definition.gd
class_name ItemDefinition
extends Resource

@export var id: StringName
@export var display_name: String

# Optional components — not all items have all features
@export var weapon_data: WeaponData      # null if not a weapon
@export var consumable_data: ConsumableData
@export var equipment_data: EquipmentData

func is_weapon() -> bool:
    return weapon_data != null

func is_consumable() -> bool:
    return consumable_data != null
```

```gdscript
# weapon_data.gd (NOT a full item, just weapon aspects)
class_name WeaponData
extends Resource

@export var damage: int
@export var attack_speed: float
```

This lets you have items that are BOTH weapon AND consumable (throwing potions?) without multiple inheritance.

---

## Common Patterns

### Spawning from Definition

```gdscript
func spawn_item(id: StringName, position: Vector3) -> void:
    var instance := ItemDatabase.create_instance(id)
    var node := preload("res://scenes/world_item.tscn").instantiate()
    node.setup(instance)
    node.global_position = position
    world.add_child(node)
```

### Picking Up (Instance Survives Node)

```gdscript
# world_item.gd
extends Node3D

var item: ItemInstance

func interact() -> void:
    if GameSystems.inventory.add(item):
        queue_free()  # Node dies, instance lives on in inventory
```

### Dropping (Instance Gets New Node)

```gdscript
func drop_item(instance: ItemInstance, position: Vector3) -> void:
    GameSystems.inventory.remove(instance)
    var node := preload("res://scenes/world_item.tscn").instantiate()
    node.setup(instance)  # Same instance, new node
    node.global_position = position
    world.add_child(node)
```

### Loot Tables

```gdscript
# loot_table.gd
class_name LootTable
extends Resource

@export var entries: Array[LootEntry] = []

func roll() -> Array[ItemInstance]:
    var results: Array[ItemInstance] = []
    for entry in entries:
        if randf() <= entry.chance:
            var qty := randi_range(entry.min_quantity, entry.max_quantity)
            results.append(ItemDatabase.create_instance(entry.item_id, qty))
    return results
```

```gdscript
# loot_entry.gd
class_name LootEntry
extends Resource

@export var item_id: StringName
@export var chance: float = 1.0
@export var min_quantity: int = 1
@export var max_quantity: int = 1
```

---

## Gotchas and Pitfalls

### 1. Modifying Definitions at Runtime

```gdscript
# CATASTROPHIC — changes affect ALL instances AND persist in editor
iron_axe_definition.damage = 999

# CORRECT — modify the instance's bonus, not the definition
iron_axe_instance.bonus_damage = 10
```

### 2. Shallow Duplicate

```gdscript
# BAD — nested array is shared reference
var copy := original_instance.duplicate()
copy.enchantments.append(new_enchant)  # Mutates BOTH!

# GOOD — deep duplicate
var copy := original_instance.duplicate(true)

# BETTER — explicit reconstruction
var copy := ItemInstance.new(original.definition, original.quantity)
copy.durability = original.durability
copy.enchantments = original.enchantments.duplicate()
```

### 3. Missing ID Field

```gdscript
# BAD — using resource path as identifier
save_data["weapon"] = weapon.definition.resource_path
# Breaks if you reorganize folders

# GOOD — explicit stable ID
save_data["weapon_id"] = weapon.definition.id
```

### 4. Forgetting @export

```gdscript
class_name ItemInstance extends Resource

@export var definition: ItemDefinition
@export var durability: int
var temporary_buff: float  # NOT EXPORTED — won't save!
```

### 5. Constructor Parameters

```gdscript
# BAD — Godot can't instantiate this for loading
func _init(def: ItemDefinition, qty: int) -> void:
    definition = def
    quantity = qty

# GOOD — defaults allow parameterless construction
func _init(def: ItemDefinition = null, qty: int = 1) -> void:
    if def:
        definition = def
        quantity = qty
```

### 6. Type Confusion

```gdscript
# Definition: "What is the MAXIMUM durability of iron axes?"
# Instance: "What is THIS axe's CURRENT durability?"

# WRONG mental model
"iron_axe.durability = 47"  # Definitions don't have current durability!

# CORRECT mental model
"iron_axe_definition.max_durability = 100"  # Template says max is 100
"player_axe_instance.durability = 47"       # This specific axe is at 47
```

---

## Checklist

- [ ] Data designed BEFORE code written
- [ ] Definitions are `Resource` scripts with `@export` vars
- [ ] Definitions are `.tres` files in `res://data/`
- [ ] Definitions are NEVER modified at runtime
- [ ] Instances exist only when data outlives nodes
- [ ] Instances hold reference to definition (not copy)
- [ ] Instances also extend `Resource` (for serialization)
- [ ] All saveable fields have `@export`
- [ ] Database uses `ResourceLoader.list_directory()` (not `DirAccess`)
- [ ] Load uses `CACHE_MODE_IGNORE`
- [ ] IDs are `StringName` for performance

---

## The Payoff

When you internalize data-driven design:

1. **Designers edit `.tres` files, not code** — faster iteration, fewer bugs
2. **Save system is trivial** — Resources serialize themselves
3. **New content = new data files** — no code changes for new items
4. **Logic is testable** — pass any definition to any function
5. **Memory efficient** — thousands of instances, handful of definitions
6. **Reasoning is clear** — "what can it be?" vs "what is it now?" never confused

> The Ultimate Test Can someone create a new item type by **only** creating a `.tres` file?
> 
> If yes, you've achieved data-driven design. If no, you've embedded content in code.