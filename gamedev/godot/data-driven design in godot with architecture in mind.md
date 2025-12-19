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


# immutable config vs mutable saving state of progress

static data like the template, definition, during design time is stored in `res://` is immutable, while saving state or just mutable data should be in the `user://`


## **`res://`**
- This is your **project’s resources path**.
- It points to the **read-only bundle of files shipped with your game** (all your .tres, .scn, .gd, textures, sounds, etc.).
- When you run in the editor, res:// maps to your project folder on disk.
    When you export the game, everything in res:// is packed into a .pck or .zip file.
    On a player’s machine, that package is **read-only**.
- If you try to write a save file into res://, it’ll work in the editor (since it’s just your filesystem), but fail or be ignored in a built game — because the player doesn’t have write permission to your packed assets.

**Usage:** store only **static, authored assets**. Config .tres, .tscn, art, sound, scripts. Immutable design data.

---

## **`user://`**
- This is the **per-user writable data folder** Godot provides.
- Path differs by platform:
    - Windows: %APPDATA%/Godot/app_userdata/YourGameName/
    - macOS: ~/Library/Application Support/Godot/app_userdata/YourGameName/
    - Linux: ~/.local/share/godot/app_userdata/YourGameName/
- On export, this is where you can **create, write, and update files at runtime**.
- Players always have write permission here.
- This is your proper place for:
    - Save games
    - Player profiles
    - Config settings (keybindings, graphics options)
    - Logs, caches, etc.

**Usage:** everything **mutable at runtime** that must persist between sessions.



TLDR:
- DeckConfig (**Resource**, in `res://`): editor-authored list of **CardConfig ids** (or weighted entries or `.tres` resources created from `.gd` resource class defenition) for starters/recipes.
- ProfileSave (**Resource**, written to `user://`) for persistent ownership/progress.
- *for the local scoped like current runtime state of the board game states like health of the card just a regular script with fields in it*

---

# Data-Driven Design in Godot 4

A reference for separating data from presentation, handling mutable state, and implementing save systems.
By my experience this is way better than cluttered mess without architecture in mind

---

## Core Concept

**[[data-driven design]]** separates *data* from *code* from *visuals*.


### The Philosophy

> "Define your data first. Keep definitions separate from runtime logic. Keep runtime simulation separate from presentation. Systems manipulate structured data, and the view layer only displays the results."

### Three Layers

```
┌─────────────────────────────────────────────────────────────────┐
│                          DATA                                   │
│  Source of truth. Designed first. Lives in one place.           │
│  ┌─────────────────┐     ┌─────────────────┐                    │
│  │   Definition    │     │    Instance     │                    │
│  │   (template)    │────▶│   (runtime)     │                    │
│  │                 │     │                 │                    │
│  │ - id            │     │ - definition    │                    │
│  │ - name          │     │ - durability    │                    │
│  │ - max_durability│     │ - quantity      │                    │
│  └─────────────────┘     └─────────────────┘                    │
│       (shared)               (unique)                           │
└─────────────────────────────────────────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────────┐
│                         SYSTEMS                                 │
│  Operate on data. Contain game logic. Don't know about visuals. │
│                                                                 │
│  - InventorySystem.add(instance)                                │
│  - CombatSystem.apply_damage(weapon_instance, target)           │
│  - CraftingSystem.craft(recipe, ingredients)                    │
└─────────────────────────────────────────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────────┐
│                       ART / VIEWS                               │
│  Read data. Display it. Never modify game state directly.       │
│                                                                 │
│  - WorldItem (3D mesh that reads instance.definition.mesh)      │
│  - InventorySlot (UI that reads instance.texture, durability)   │
│  - HealthBar (reads player_data.health)                         │
└─────────────────────────────────────────────────────────────────┘
```

### Why This Order?

1. **Data first** — Forces you to understand the problem before coding solutions
2. **Systems second** — Logic that *transforms* data, as well as stores it runtime, testable without visuals. Note For self-contained, ephemeral, single-scene mechanics, direct manipulation is fine. It is also a centralized for the *system*, the users. no need to spread a mess
3. **Views last** — Just render current state, trivially replaceable

---

## The Two Types of Data

|Type|Purpose|Mutability|Storage|
|---|---|---|---|
|**Definition**|Template/prototype describing _what_ something is|Read-only at runtime|`.tres` files in `res://`|
|**Instance**|Specific occurrence with runtime state|Mutable|Created at runtime, saved to `user://`|

### Why Separate Them?

- **Memory**: 100 iron axes share one `IronAxeDefinition`, not 100 copies
- **Saving**: Only serialize what changed (durability: 47), not the entire definition
- **Editing**: Change `IronAxeDefinition.damage` once, all axes update
- **Clarity**: Obvious what's static vs dynamic

---

## Design Process: Data First

Before writing any code, answer these questions:

### 1. What data exists?

List every piece of information your feature needs:

```
Item:
  - id (unique identifier)
  - name (display string)
  - description
  - icon texture
  - max durability
  - weight
  - stackable?
  - max stack size
```

### 2. What's static vs dynamic?

Separate into definition (never changes) and instance (changes at runtime):

| Static (Definition) | Dynamic (Instance) |
| ------------------- | ------------------ |
| id                  | current durability |
| name                | quantity           |
| max durability      | custom name        |
| icon                | enchantments       |
| base damage         | position in world  |

- Definition = static domain knowledge (what IS a goblin)
- Instance = runtime domain knowledge (what is THIS goblin's state right now)

### 3. What operations happen on this data?

List the verbs — these become your system functions:

```
- pick_up(instance) → add to inventory
- drop(instance, position) → spawn in world
- use(instance) → apply effect, reduce quantity
- damage(instance, amount) → reduce durability
- repair(instance) → restore durability
- split_stack(instance, amount) → create new instance
- merge_stacks(a, b) → combine quantities
```

### 4. What needs to display this data?

List the views — these just read and render:

```
- WorldItem: 3D mesh + collision, shows instance in world
- InventorySlot: icon + quantity label + durability bar
- Tooltip: name + description + stats
- EquipmentSlot: specialized slot for worn items
```

Now you can implement. The structure is already clear.

---

## Implementation

### Definition (Template)

Authored in editor, never modified at runtime.

```gdscript
# item_definition.gd
class_name ItemDefinition extends Resource

@export var id: StringName
@export var display_name: String
@export var description: String
@export var texture: Texture2D
@export var max_durability: int = 100
@export var max_stack: int = 1
@export var weight: float = 1.0
```

Create instances in FileSystem: `Right-click → New Resource → ItemDefinition`

Save as: `res://data/items/iron_axe.tres`

### Instance (Runtime State)

Created when spawning/picking up, holds mutable state.

```gdscript
# item_instance.gd
class_name ItemInstance extends Resource


# for read-only. reason to store in ItemDefinition and not to copy, is because it is reference, and we don't need to repeat ourselves 100x times
@export var definition: ItemDefinition

# for modifications
@export var durability: int = -1
@export var quantity: int = 1
@export var custom_data: Dictionary = {}

# Constructor
# NOTE: Godot requires that the `_init()` method for Resources, and other `Object`-derived types that are instantiated by the engine (like Nodes), either be parameterless or have default values for all its parameters.
func _init(def: ItemDefinition = null, qty: int = 1) -> void:
    if def:
        definition = def
        durability = def.max_durability
        quantity = qty

# Convenience accessors (read from definition)
var display_name: String:
    get: return definition.display_name if definition else ""

var texture: Texture2D:
    get: return definition.texture if definition else null

# State modification
func damage(amount: int) -> bool:
    durability -= amount
    return durability <= 0  # returns true if broken
```

### System (Logic)

Operates on data. Contains game rules. Doesn't know about nodes or visuals.

```gdscript
# inventory_system.gd
class_name InventorySystem extends RefCounted

var items: Array[ItemInstance] = []
var capacity: int = 20

signal item_added(item: ItemInstance, index: int)
signal item_removed(item: ItemInstance, index: int)
signal item_changed(item: ItemInstance, index: int)


func add(item: ItemInstance) -> bool:
    # Try stacking first
    if item.definition.max_stack > 1:
        for i in items.size():
            if _can_stack(items[i], item):
                items[i].quantity += item.quantity
                item_changed.emit(items[i], i)
                return true
    
    # Find empty slot
    if items.size() < capacity:
        items.append(item)
        item_added.emit(item, items.size() - 1)
        return true
    
    return false  # inventory full


func remove(index: int) -> ItemInstance:
    if index < 0 or index >= items.size():
        return null
    var item := items[index]
    items.remove_at(index)
    item_removed.emit(item, index)
    return item


func use(index: int) -> void:
    var item := items[index]
    if not item:
        return
    
    # Delegate to item-specific logic
    match item.definition.item_type:
        ItemType.CONSUMABLE:
            _consume(item, index)
        ItemType.EQUIPMENT:
            _equip(item, index)


func _can_stack(a: ItemInstance, b: ItemInstance) -> bool:
    if not a or not b:
        return false
    if a.definition != b.definition:
        return false
    return a.quantity + b.quantity <= a.definition.max_stack


func _consume(item: ItemInstance, index: int) -> void:
    # Apply effect (emit signal for other systems to react)
    consumed.emit(item)
    
    item.quantity -= 1
    if item.quantity <= 0:
        items.remove_at(index)
        item_removed.emit(item, index)
    else:
        item_changed.emit(item, index)


signal consumed(item: ItemInstance)
```

```gdscript
# combat_system.gd
class_name CombatSystem extends RefCounted

signal damage_dealt(attacker, target, amount: int)
signal weapon_degraded(weapon: ItemInstance)


func attack(attacker_weapon: ItemInstance, target_health: HealthData) -> void:
    if not attacker_weapon or not target_health:
        return
    
    var damage: int = _calculate_damage(attacker_weapon)
    target_health.current -= damage
    damage_dealt.emit(attacker_weapon, target_health, damage)
    
    # Degrade weapon
    attacker_weapon.durability -= 1
    weapon_degraded.emit(attacker_weapon)


func _calculate_damage(weapon: ItemInstance) -> int:
    var base: int = weapon.definition.damage
    var modifier: float = weapon.durability / float(weapon.definition.max_durability)
    return int(base * modifier)
```

**Key points:**

- Systems are pure logic — no `Node`, no `@onready`, no `$Path`
- Communicate through signals (views subscribe to these)
- Take data in, modify data, emit what happened
- Can be `RefCounted` (owned by something) or `Autoload` (global)

---

### Node (View)

Displays an instance. Temporary — can be created/destroyed freely. **Reads data, doesn't contain logic.**

```gdscript
# world_item.gd
class_name WorldItem extends Node3D

var item: ItemInstance

func setup(instance: ItemInstance) -> void:
    item = instance
    _update_visuals()

func _update_visuals() -> void:
    if not item:
        return
    $MeshInstance3D.mesh = load(item.definition.mesh_path)
    # etc.

func _on_picked_up(inventory: InventorySystem) -> void:
    inventory.add(item)  # call system, pass data
    queue_free()         # node dies, instance lives on
```

```gdscript
# inventory_ui.gd
class_name InventoryUI extends Control

@export var inventory: InventorySystem
var slots: Array[InventorySlot] = []


func _ready() -> void:
    # Subscribe to system signals
    inventory.item_added.connect(_on_item_added)
    inventory.item_removed.connect(_on_item_removed)
    inventory.item_changed.connect(_on_item_changed)
    
    _rebuild_slots()


func _on_item_added(item: ItemInstance, index: int) -> void:
    slots[index].setup(item)


func _on_item_removed(item: ItemInstance, index: int) -> void:
    slots[index].clear()
    _rebuild_slots()  # reflow if needed


func _on_item_changed(item: ItemInstance, index: int) -> void:
    slots[index].refresh()
```

```gdscript
# inventory_slot.gd
class_name InventorySlot extends PanelContainer

var item: ItemInstance

func setup(instance: ItemInstance) -> void:
    item = instance
    refresh()

func refresh() -> void:
    if item:
        $Icon.texture = item.texture
        $Quantity.text = str(item.quantity) if item.quantity > 1 else ""
        $DurabilityBar.value = float(item.durability) / item.definition.max_durability
        $DurabilityBar.visible = item.definition.max_durability > 0
    else:
        clear()

func clear() -> void:
    item = null
    $Icon.texture = null
    $Quantity.text = ""
    $DurabilityBar.visible = false
```

**Key points:**

- Views subscribe to system signals
- When system emits `item_changed`, view calls `refresh()`
- Views never modify data directly — they call system methods
- One-way data flow: `System modifies Data → emits signal → View reads Data`

---

## Database Pattern for definition management

Other patterns: [[Godot definition management approaches]]

Central registry for looking up definitions by ID.

```gdscript
# item_database.gd
class_name ItemDatabase extends Node  # Autoload

var _items: Dictionary = {}  # id → ItemDefinition

func _ready() -> void:
    _load_all_items()

func _load_all_items() -> void:
    var dir := DirAccess.open("res://data/items/")
    if dir:
        dir.list_dir_begin()
        var file_name := dir.get_next()
        while file_name != "":
            if file_name.ends_with(".tres"):
                var item: ItemDefinition = load("res://data/items/" + file_name)
                _items[item.id] = item
            file_name = dir.get_next()

func get_definition(id: StringName) -> ItemDefinition:
    return _items.get(id)

func create_instance(id: StringName, quantity: int = 1) -> ItemInstance:
    var def := get_definition(id)
    if def:
        return ItemInstance.new(def, quantity)
    return null
```

[[data access in godot]]

Usage when needed (view, systems):

```gdscript
var axe := ItemDatabase.create_instance(&"iron_axe")
inventory.add(axe)
```

---


## What Manages What

| Pattern      | Manages     | Lifetime                               |
| ------------ | ----------- | -------------------------------------- |
| **Database** | Definitions | Entire game (loaded once)              |
| **System**   | Instances   | Runtime (created, modified, destroyed) |


## Save System

### Strategy

1. Instances extend `Resource` → Godot handles serialization
2. Wrap all saveable state in a single `SaveGame` resource
3. Definitions are referenced by path, auto-resolved on load

### SaveGame Resource

```gdscript
# save_game.gd
class_name SaveGame extends Resource

@export var player_position: Vector3
@export var player_health: int
@export var inventory_items: Array[ItemInstance] = []
@export var world_items: Array[WorldItemData] = []
@export var quest_states: Dictionary = {}
@export var play_time_seconds: float = 0.0
```

### Save Manager

```gdscript
# save_manager.gd
class_name SaveManager extends Node  # Autoload

const SAVE_PATH := "user://savegame.tres"

func save_game() -> void:
    var save := SaveGame.new()
    
    # Collect state from game systems
    save.player_position = player.global_position
    save.player_health = player.health
    save.inventory_items = player.inventory.get_all_items()
    save.world_items = _collect_world_items()
    save.quest_states = QuestManager.get_states()
    save.play_time_seconds = Time.get_ticks_msec() / 1000.0
    
    # Save
    var error := ResourceSaver.save(save, SAVE_PATH)
    if error != OK:
        push_error("Failed to save: %s" % error)

func load_game() -> void:
    if not ResourceLoader.exists(SAVE_PATH):
        push_warning("No save file found")
        return
    
    var save: SaveGame = ResourceLoader.load(SAVE_PATH)
    
    # Restore state
    player.global_position = save.player_position
    player.health = save.player_health
    player.inventory.set_items(save.inventory_items)
    _spawn_world_items(save.world_items)
    QuestManager.set_states(save.quest_states)

func _collect_world_items() -> Array[WorldItemData]:
    var result: Array[WorldItemData] = []
    for node in get_tree().get_nodes_in_group("world_items"):
        var data := WorldItemData.new()
        data.item = node.item
        data.position = node.global_position
        result.append(data)
    return result

func _spawn_world_items(items: Array[WorldItemData]) -> void:
    for data in items:
        var node := preload("res://scenes/world_item.tscn").instantiate()
        node.setup(data.item)
        node.global_position = data.position
        world.add_child(node)
```

### Helper for World Items

```gdscript
# world_item_data.gd
class_name WorldItemData extends Resource

@export var item: ItemInstance
@export var position: Vector3
```

> NOTE: **you should duplicate on load. No need on save.**

Here's why:

## On Save: No Duplication Needed

```gdscript
func save_game() -> void:
    var save := SaveGame.new()
    save.inventory_items = inventory.items  # Direct reference is fine
    ResourceSaver.save(save, SAVE_PATH)
```

**Why it's safe:** `ResourceSaver.save()` serializes the data immediately. It doesn't keep references to your live instances. Once saved, the `SaveGame` resource can be discarded.

## On Load: MUST Duplicate

```gdscript
# BAD - Don't do this
func load_game() -> void:
    var save: SaveGame = ResourceLoader.load(SAVE_PATH)
    inventory.items = save.inventory_items  # ❌ WRONG

# GOOD - Duplicate the array
func load_game() -> void:
    var save: SaveGame = ResourceLoader.load(SAVE_PATH)
    inventory.items = save.inventory_items.duplicate()  # ✅ CORRECT
```

**Why?** Godot caches loaded resources. If you assign directly:

1. Load save file → `save.inventory_items` points to cached array
2. Modify items during gameplay → you're modifying the _cached resource_
3. Load save again → you get the modified version, not the original

## The Deep Copy Problem

`duplicate()` is shallow by default. For nested structures:

```gdscript
# If ItemInstance contains arrays/dictionaries
inventory.items = save.inventory_items.duplicate(true)  # deep copy

# Or be explicit
func load_inventory(saved_items: Array[ItemInstance]) -> void:
    inventory.items.clear()
    for item in saved_items:
        var new_item := ItemInstance.new(item.definition, item.quantity)
        new_item.durability = item.durability
        new_item.custom_data = item.custom_data.duplicate(true)
        inventory.items.append(new_item)
```

## Better Pattern: Use CACHE_MODE_IGNORE on Load

```gdscript
func load_game() -> void:
    var save: SaveGame = ResourceLoader.load(
        SAVE_PATH, 
        "", 
        ResourceLoader.CACHE_MODE_IGNORE
    )
    inventory.items = save.inventory_items  # Now safe - fresh copy every time
```
This bypasses the cache entirely. Each load creates a new resource tree.

---

## How Definition References Survive Save/Load

When Godot saves an `ItemInstance`, it stores the `definition` property as a resource path:

```
[resource]
definition = ExtResource("res://data/items/iron_axe.tres")
durability = 47
quantity = 1
```

On load, Godot:

1. Sees `ExtResource("res://data/items/iron_axe.tres")`
2. Loads that definition (or returns cached version)
3. Assigns it to `instance.definition`

**Result**: Instances automatically reconnect to their shared definitions.

---

## Alternative: Manual Serialization

If you need more control (binary format, encryption, JSON export):

```gdscript
# item_instance.gd (add these methods)

func to_dict() -> Dictionary:
    return {
        "def_id": definition.id,
        "durability": durability,
        "quantity": quantity,
        "custom": custom_data,
    }

static func from_dict(data: Dictionary, db: ItemDatabase) -> ItemInstance:
    var def := db.get_definition(data["def_id"])
    if not def:
        push_error("Unknown item: %s" % data["def_id"])
        return null
    
    var inst := ItemInstance.new(def, data["quantity"])
    inst.durability = data["durability"]
    inst.custom_data = data.get("custom", {})
    return inst
```

```gdscript
# Binary save with store_var
func save_game_binary() -> void:
    var data := {
        "version": 1,
        "player": {"pos": player.global_position, "health": player.health},
        "inventory": player.inventory.get_all_items().map(func(i): return i.to_dict()),
    }
    
    var file := FileAccess.open("user://save.dat", FileAccess.WRITE)
    file.store_var(data)

func load_game_binary() -> void:
    var file := FileAccess.open("user://save.dat", FileAccess.READ)
    var data: Dictionary = file.get_var()
    
    player.global_position = data["player"]["pos"]
    player.health = data["player"]["health"]
    
    for item_data in data["inventory"]:
        var inst := ItemInstance.from_dict(item_data, ItemDatabase)
        player.inventory.add(inst)
```

---

## Common Patterns

### Spawning Items in World

```gdscript
# From definition ID
func spawn_item(id: StringName, pos: Vector3) -> void:
    var instance := ItemDatabase.create_instance(id)
    var node := preload("res://scenes/world_item.tscn").instantiate()
    node.setup(instance)
    node.global_position = pos
    world.add_child(node)

# From existing instance (e.g., dropping from inventory)
func drop_item(instance: ItemInstance, pos: Vector3) -> void:
    var node := preload("res://scenes/world_item.tscn").instantiate()
    node.setup(instance)
    node.global_position = pos
    world.add_child(node)
```

### Picking Up Items

```gdscript
# world_item.gd
func _on_interaction_area_body_entered(body: Node3D) -> void:
    if body.is_in_group("player"):
        if body.inventory.add(item):
            queue_free()
```

### Using/Consuming Items

```gdscript
# inventory.gd
func use_item(index: int) -> void:
    var item := items[index]
    if not item:
        return
    
    match item.definition.item_type:
        ItemType.CONSUMABLE:
            _apply_consumable_effect(item)
            item.quantity -= 1
            if item.quantity <= 0:
                items[index] = null
        
        ItemType.EQUIPMENT:
            _equip_item(item)
```

### Checking Item Properties

```gdscript
# Access definition (static) properties
var damage: int = weapon.definition.damage
var name: String = weapon.definition.display_name

# Access instance (dynamic) properties  
var current_durability: int = weapon.durability
var is_broken: bool = weapon.durability <= 0
```

---

## Extending the Pattern

### Inheritance for Item Types

```gdscript
# weapon_definition.gd
class_name WeaponDefinition extends ItemDefinition

@export var damage: int
@export var attack_speed: float
@export var weapon_type: WeaponType
```

```gdscript
# consumable_definition.gd
class_name ConsumableDefinition extends ItemDefinition

@export var heal_amount: int
@export var buff_type: BuffType
@export var buff_duration: float
```

### Instance-Specific State

```gdscript
# For weapons with random stats, enchantments, etc.
class_name WeaponInstance extends ItemInstance

@export var enchantments: Array[Enchantment] = []
@export var bonus_damage: int = 0

var total_damage: int:
    get: 
        var base: int = (definition as WeaponDefinition).damage
        return base + bonus_damage + _enchantment_bonus()
```

---

## Quick Reference

|Want to...|Do this|
|---|---|
|Create new item type|New `Resource` script extending `ItemDefinition`|
|Add runtime state|Add `@export var` to `ItemInstance`|
|Spawn item in world|`ItemDatabase.create_instance(id)` → node.setup(instance)|
|Pick up item|Pass `instance` to inventory, `queue_free()` the node|
|Save game|`ResourceSaver.save(save_game, path)`|
|Load game|`ResourceLoader.load(path)`|
|Look up definition|`ItemDatabase.get_definition(id)`|

---

## Checklist

- [ ] Definitions are `Resource` scripts with `@export` vars
- [ ] Definitions are `.tres` files in `res://data/`
- [ ] Instances hold reference to definition + mutable state
- [ ] Instances also extend `Resource` (for easy saving)
- [ ] Nodes hold instance reference, not definition
- [ ] Database autoload for looking up definitions by ID
- [ ] SaveGame resource wraps all saveable state
- [ ] Save/load through `ResourceSaver`/`ResourceLoader`

---

## Gotchas

1. **Don't modify definitions at runtime** — changes affect all instances and persist across sessions in editor
2. **`duplicate()` has issues** — nested Arrays/Dictionaries may not deep-copy; use explicit instance creation instead
3. **Resource caching** — Godot caches loaded resources; use `ResourceLoader.load(path, "", ResourceLoader.CACHE_MODE_IGNORE)` if you need fresh copies
4. **Export vars required for saving** — only `@export` properties are serialized by `ResourceSaver`
5. **Circular references** — avoid instances referencing each other in ways that create cycles; Godot handles it but it complicates debugging


### 1. Where instances live and how SaveManager accesses them

Systems hold the instances. SaveManager gets them by **directly referencing the systems** (usually via Autoload or dependency injection):

```gdscript
# Systems are Autoloads or injected references
# SaveManager.gd

@onready var inventory: InventorySystem = GameSystems.inventory
@onready var quest_system: QuestSystem = GameSystems.quests
@onready var world_state: WorldStateSystem = GameSystems.world

func save_game() -> void:
    var save := SaveGame.new()
    
    # Directly pull data from systems
    save.inventory_items = inventory.items
    save.quest_states = quest_system.states
    save.world_items = world_state.dropped_items
    
    ResourceSaver.save(save, SAVE_PATH)

func load_game() -> void:
    var save: SaveGame = ResourceLoader.load(SAVE_PATH)
    
    # Directly push data back into systems
    inventory.items = save.inventory_items
    quest_system.states = save.quest_states
    world_state.dropped_items = save.world_items
    
    # Systems emit signals, views update automatically
```

Or systems expose getter methods:

```gdscript
# inventory_system.gd
func get_all_items() -> Array[ItemInstance]:
    return items.duplicate()

func set_all_items(new_items: Array[ItemInstance]) -> void:
    items = new_items
    inventory_rebuilt.emit()  # views refresh
```

### 2. Dependency direction

```
        CALLS (downward)              SIGNALS (upward)
        ─────────────────►            ◄─────────────────

┌─────────────────────────────────────────────────────────────┐
│                         VIEWS                               │
│                                                             │
│   inventory_ui.gd                                           │
│   ├── has reference to: InventorySystem                     │
│   ├── calls: inventory.add(item), inventory.use(index)      │
│   └── listens to: item_added, item_removed, item_changed    │
│                                                             │
└──────────────────────────┬──────────────────────────────────┘
                           │ calls ▼         ▲ signals
┌──────────────────────────▼──────────────────────────────────┐
│                        SYSTEMS                              │
│                                                             │
│   inventory_system.gd                                       │
│   ├── owns: Array[ItemInstance]                             │
│   ├── modifies: item.durability, item.quantity              │
│   └── emits: item_added, item_changed, item_removed         │
│                                                             │
└──────────────────────────┬──────────────────────────────────┘
                           │ owns/modifies
┌──────────────────────────▼──────────────────────────────────┐
│                          DATA                               │
│                                                             │
│   ItemInstance (durability, quantity, custom_data)          │
│   └── references: ItemDefinition (static template)          │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**Rules:**

- Views **know about** Systems (hold references, call methods)
- Systems **don't know about** Views (only emit signals blindly)
- Systems **own and modify** Data
- Data **knows nothing** (pure state)

This means you can:

- Swap UI completely without touching systems
- Run systems in tests without any nodes
- Add new views that subscribe to same signals


[[Systems access approaches in Godot]]


Advice:
- if something is complicated try to boil it down to something easy and helpful like a new system or manager for the global agnostic player speed and position tracking


# Data separation level

The pattern isn't universal—it solves specific problems.

**Definition-only**

Static template data extracted into a Resource.

```gdscript
# EnemyDefinition resource
@export var max_health := 100
@export var damage := 25

# Enemy node
@export var definition: EnemyDefinition
var health: int  # mutable state lives on the node

func _ready():
    health = definition.max_health
```

Benefits:

- Data editable without touching code
- Shared across scenes (all goblins use `goblin.tres`)
- Queryable without instantiation

Use when: you want clean separation, even if the node is the source of truth at runtime.

---

**Definition + Instance**

Static template _plus_ a separate Resource for mutable runtime state.

```gdscript
# ItemDefinition (static)
# ItemInstance (mutable, references definition)

# Node just displays the instance
```

Additional benefits:

- Thing exists without a node (inventory, save file)
- Mutable state survives node destruction
- Multiple views of same instance stay in sync

Use when: the data's lifecycle is independent of any node.

---

**Decision:**

| Question                             | Yes →                   |
| ------------------------------------ | ----------------------- |
| Want data out of code?               | Definition-only minimum |
| Does mutable state outlive the node? | Need Instance layer     |

The first is almost always worth it. The second depends on the problem.



