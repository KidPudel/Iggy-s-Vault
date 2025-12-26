# Data-Driven Design in Godot 4

A reference for separating data from presentation, handling mutable state, and implementing save systems.

- [[#When to Use This Pattern|When to Use This Pattern]]
- [[#Core Concept|Core Concept]]
	- [[#Core Concept#The Philosophy|The Philosophy]]
	- [[#Core Concept#Three Layers|Three Layers]]
	- [[#Core Concept#Why This Order?|Why This Order?]]
	- [[#Core Concept#System Ownership|System Ownership]]
- [[#The Two Types of Data|The Two Types of Data]]
	- [[#The Two Types of Data#Why Separate Them?|Why Separate Them?]]
- [[#Storage Paths|Storage Paths]]
	- [[#Storage Paths#`res://` — Project Resources (Immutable)|`res://` — Project Resources (Immutable)]]
	- [[#Storage Paths#`user://` — User Data (Mutable)|`user://` — User Data (Mutable)]]
- [[#Design Process: Data First|Design Process: Data First]]
	- [[#Design Process: Data First#1. What data exists?|1. What data exists?]]
	- [[#Design Process: Data First#2. What's static vs dynamic?|2. What's static vs dynamic?]]
	- [[#Design Process: Data First#3. What operations happen on this data?|3. What operations happen on this data?]]
	- [[#Design Process: Data First#4. What displays this data?|4. What displays this data?]]
- [[#Implementation|Implementation]]
	- [[#Implementation#Definition (Template)|Definition (Template)]]
	- [[#Implementation#Instance (Runtime State)|Instance (Runtime State)]]
	- [[#Implementation#System (Logic)|System (Logic)]]
	- [[#Implementation#View (Node)|View (Node)]]
- [[#Database Pattern|Database Pattern]]
	- [[#Database Pattern#Option 1: ResourceLoader.list_directory() (Recommended)|Option 1: ResourceLoader.list_directory() (Recommended)]]
	- [[#Database Pattern#Option 2: Explicit Export Array|Option 2: Explicit Export Array]]
- [[#Systems vs Managers (Layer 0 vs Layer 1)|Systems vs Managers (Layer 0 vs Layer 1)]]
	- [[#Systems vs Managers (Layer 0 vs Layer 1)#Layer 0: Dumb Systems|Layer 0: Dumb Systems]]
	- [[#Systems vs Managers (Layer 0 vs Layer 1)#Layer 1: Smart Orchestration|Layer 1: Smart Orchestration]]
	- [[#Systems vs Managers (Layer 0 vs Layer 1)#Micro-Managers: Technical Complexity Isolation|Micro-Managers: Technical Complexity Isolation]]
	- [[#Systems vs Managers (Layer 0 vs Layer 1)#Wishlist-First Design|Wishlist-First Design]]
	- [[#Systems vs Managers (Layer 0 vs Layer 1)#The Architecture Diagram|The Architecture Diagram]]
	- [[#Systems vs Managers (Layer 0 vs Layer 1)#Where Game Logic Lives|Where Game Logic Lives]]
	- [[#Systems vs Managers (Layer 0 vs Layer 1)#Communication Rules|Communication Rules]]
	- [[#Systems vs Managers (Layer 0 vs Layer 1)#When Should Views Call Layer 0 vs Layer 1?|When Should Views Call Layer 0 vs Layer 1?]]
	- [[#Systems vs Managers (Layer 0 vs Layer 1)#Layer 1 to Layer 1 Communication|Layer 1 to Layer 1 Communication]]
	- [[#Systems vs Managers (Layer 0 vs Layer 1)#When to Split a Manager|When to Split a Manager]]
- [[#Save System|Save System]]
	- [[#Save System#Strategy|Strategy]]
	- [[#Save System#SaveGame Resource|SaveGame Resource]]
	- [[#Save System#Save Manager|Save Manager]]
	- [[#Save System#How Definition References Survive Save/Load|How Definition References Survive Save/Load]]
	- [[#Save System#Save: No Duplication Needed|Save: No Duplication Needed]]
	- [[#Save System#Load: Use CACHE_MODE_IGNORE|Load: Use CACHE_MODE_IGNORE]]
- [[#Manual Serialization Alternative|Manual Serialization Alternative]]
- [[#Common Patterns|Common Patterns]]
	- [[#Common Patterns#Spawning Items|Spawning Items]]
	- [[#Common Patterns#Picking Up Items|Picking Up Items]]
- [[#Extending the Pattern|Extending the Pattern]]
	- [[#Extending the Pattern#Inheritance for Item Types|Inheritance for Item Types]]
	- [[#Extending the Pattern#Instance-Specific State|Instance-Specific State]]
- [[#Data Separation Levels|Data Separation Levels]]
	- [[#Data Separation Levels#Definition-Only|Definition-Only]]
	- [[#Data Separation Levels#Definition + Instance|Definition + Instance]]
- [[#Quick Reference|Quick Reference]]
- [[#Checklist|Checklist]]
- [[#Gotchas|Gotchas]]
- [[#Adapting for Small Projects|Adapting for Small Projects]]
	- [[#Adapting for Small Projects#What to Keep|What to Keep]]
	- [[#Adapting for Small Projects#What to Skip|What to Skip]]
	- [[#Adapting for Small Projects#Lightweight Database|Lightweight Database]]
	- [[#Adapting for Small Projects#Lightweight Save System|Lightweight Save System]]
	- [[#Adapting for Small Projects#When to Upgrade|When to Upgrade]]
	- [[#Adapting for Small Projects#Jam-Friendly Structure|Jam-Friendly Structure]]
	- [[#Adapting for Small Projects#The Minimum Principle|The Minimum Principle]]


---

## When to Use This Pattern

This architecture pays off when:

- Data exists outside nodes (inventory, saves, scene transfers)
- You have many similar entities (50+ item types, enemy variants)
- Multiple systems interact with the same data
- You need testable logic without running scenes

**Skip it** for jam games, single-scene puzzles, or prototypes. For self-contained, ephemeral mechanics, direct node manipulation is fine. See [[#Adapting for Small Projects]] for a lightweight approach that keeps the benefits without the ceremony.

---

## Core Concept

[[data-driven design]] separates _data_ from _code_ from _visuals_.

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
│  Operate on data. Don't know about visuals or game context.     │
│                                                                 │
│  - InventorySystem.add(instance)                                │
│  - HealthSystem.take_damage(entity, amount)                     │
│  - TurnSystem.next_turn()                                       │
└─────────────────────────────────────────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────────┐
│                       VIEWS (Nodes)                             │
│  Read data. Display it. Never modify game state directly.       │
│                                                                 │
│  - WorldItem (3D mesh that reads instance.definition.mesh)      │
│  - InventorySlot (UI that reads instance.texture, durability)   │
│  - HealthBar (reads player_data.health)                         │
└─────────────────────────────────────────────────────────────────┘
```

> **Note:** This is a simplified view. The [[#Systems vs Managers (Layer 0 vs Layer 1)]] section refines this into Layer 0 (dumb systems) and Layer 1 (smart managers that orchestrate systems).

### Why This Order?

1. **Data first** — Forces you to understand the problem before coding
2. **Systems second** — Logic that transforms data, testable without visuals
3. **Views last** — Just render current state, trivially replaceable

### System Ownership

Systems extend `RefCounted` and get garbage collected if nothing holds a reference. They must be either:

- **Owned by a Manager node** — the manager instantiates and holds the system
- **Registered as Autoloads** — global singleton lifetime

---

## The Two Types of Data

| Type           | Purpose                                 | Mutability | Storage                                |
| -------------- | --------------------------------------- | ---------- | -------------------------------------- |
| **Definition** | Template describing _what_ something is | Read-only  | `.tres` files in `res://`              |
| **Instance**   | Specific occurrence with runtime state  | Mutable    | Created at runtime, saved to `user://` |

### Why Separate Them?

- **Memory**: 100 iron axes share one `IronAxeDefinition`, not 100 copies
- **Saving**: Only serialize what changed (durability: 47), not the entire definition
- **Editing**: Change `IronAxeDefinition.damage` once, all axes update
- **Clarity**: Obvious what's static vs dynamic

---

## Storage Paths

### `res://` — Project Resources (Immutable)

- Read-only bundle shipped with your game
- In editor: maps to project folder
- On export: packed into `.pck` file, player cannot write here
- **Use for**: static assets, definitions, scenes, scripts

### `user://` — User Data (Mutable)

- Per-user writable folder
- Platform-specific paths:
    - Windows: `%APPDATA%/Godot/app_userdata/YourGameName/`
    - macOS: `~/Library/Application Support/Godot/app_userdata/YourGameName/`
    - Linux: `~/.local/share/godot/app_userdata/YourGameName/`
- **Use for**: save games, settings, profiles, logs

---

## Design Process: Data First

Before writing code, answer these questions:

### 1. What data exists?

List every piece of information:

```
Item:
  - id, name, description
  - icon texture
  - max durability, weight
  - stackable?, max stack size
```

### 2. What's static vs dynamic?

|Static (Definition)|Dynamic (Instance)|
|---|---|
|id, name|current durability|
|max durability|quantity|
|icon, base damage|enchantments|

### 3. What operations happen on this data?

These become system methods:

```
- add(instance), remove(instance)
- use(instance), damage(instance, amount)
- split_stack(instance, amount), merge_stacks(a, b)
```

### 4. What displays this data?

These become view nodes:

```
- WorldItem: 3D mesh in world
- InventorySlot: UI icon + quantity
- Tooltip: name + description
```

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

Create in FileSystem: `Right-click → New Resource → ItemDefinition` Save as: `res://data/items/iron_axe.tres`

### Instance (Runtime State)

Created when spawning/picking up. Holds mutable state, references its definition.

```gdscript
# item_instance.gd
class_name ItemInstance extends Resource

@export var definition: ItemDefinition
@export var durability: int = -1
@export var quantity: int = 1
@export var custom_data: Dictionary = {}

# Godot requires parameterless _init or defaults for all params
func _init(def: ItemDefinition = null, qty: int = 1) -> void:
    if def:
        definition = def
        durability = def.max_durability
        quantity = qty

# Convenience accessors
var display_name: String:
    get: return definition.display_name if definition else ""

var texture: Texture2D:
    get: return definition.texture if definition else null

func damage(amount: int) -> bool:
    durability -= amount
    return durability <= 0  # true if broken
```

### System (Logic)

Pure logic. No nodes, no visuals. Owns and manipulates instances.

```gdscript
# inventory_system.gd
class_name InventorySystem extends RefCounted

signal item_added(item: ItemInstance, index: int)
signal item_removed(item: ItemInstance, index: int)
signal item_changed(item: ItemInstance, index: int)
signal item_consumed(item: ItemInstance)

var items: Array[ItemInstance] = []
var capacity: int = 20


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
    
    return false


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
    _consume(item, index)


func _can_stack(a: ItemInstance, b: ItemInstance) -> bool:
    if not a or not b:
        return false
    if a.definition != b.definition:
        return false
    return a.quantity + b.quantity <= a.definition.max_stack


func _consume(item: ItemInstance, index: int) -> void:
    item_consumed.emit(item)
    item.quantity -= 1
    if item.quantity <= 0:
        items.remove_at(index)
        item_removed.emit(item, index)
    else:
        item_changed.emit(item, index)


func get_all_items() -> Array[ItemInstance]:
    return items.duplicate()


func set_all_items(new_items: Array[ItemInstance]) -> void:
    items = new_items
    # Views should reconnect via signals or explicit rebuild
```

**Key points:**

- Systems are pure logic—no `Node`, no `@onready`, no `$Path`
- Communicate through signals
- Can be `RefCounted` (owned by manager) or `Autoload` (global)

### View (Node)

Displays an instance. Reads data, doesn't contain logic.

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

func _on_picked_up(inventory: InventorySystem) -> void:
    inventory.add(item)
    queue_free()  # Node dies, instance lives on
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

**One-way data flow:** `System modifies Data → emits signal → View reads Data`

---

## Database Pattern

Central registry for looking up definitions by ID.

See also: [[Godot definition management approaches]]

### Option 1: ResourceLoader.list_directory() (Recommended)

Works reliably in both editor and exports. Returns original resource filenames even after packing.

```gdscript
# item_database.gd
class_name ItemDatabase extends Node  # Autoload

var _items: Dictionary = {}  # id → ItemDefinition


func _ready() -> void:
    _load_all_items()


func _load_all_items() -> void:
    var files := ResourceLoader.list_directory("res://data/items/")
    for file_name in files:
        if file_name.ends_with(".tres"):
            var item: ItemDefinition = load("res://data/items/" + file_name)
            _items[item.id] = item


func get_definition(id: StringName) -> ItemDefinition:
    return _items.get(id)


func create_instance(id: StringName, quantity: int = 1) -> ItemInstance:
    var def := get_definition(id)
    if def:
        return ItemInstance.new(def, quantity)
    push_error("Unknown item id: %s" % id)
    return null
```

### Option 2: Explicit Export Array

Manually drag definitions into the inspector. More control, but requires manual updates.

```gdscript
# item_database.gd
class_name ItemDatabase extends Node  # Autoload

@export var all_items: Array[ItemDefinition] = []

var _items: Dictionary = {}


func _ready() -> void:
    for item in all_items:
        _items[item.id] = item
```

> **Warning**: Avoid `DirAccess.open("res://...")` for scanning definitions. It works in-editor but breaks on export—files become `.import` remaps and `DirAccess` doesn't resolve them. Use `ResourceLoader.list_directory()` instead.

Usage:

```gdscript
var axe := ItemDatabase.create_instance(&"iron_axe")
inventory.add(axe)
```

|Pattern|Manages|Lifetime|
|---|---|---|
|**Database**|Definitions|Entire game (loaded once)|
|**System**|Instances|Runtime (created, modified, destroyed)|

---

## Systems vs Managers (Layer 0 vs Layer 1)

This architecture draws from Ricard Pillosu's approach (20+ years AAA, currently building Starfinder in Godot). The key insight: **systems should be dumb, managers should be smart**.

Source: [Jonas Tyroller Podcast — "Industry Veteran About Coding for Games"](https://www.youtube.com/watch?v=Fxea6TG0PXk) (1:02:43 - 1:20:48)

|Layer|Type|Extends|Role|
|---|---|---|---|
|**Layer 0**|System|RefCounted|Dumb. Data & state only. Zero game context.|
|**Layer 1**|Manager|Node|Smart. Orchestrates systems. Contains game-specific flow.|

### Layer 0: Dumb Systems

A Layer 0 system:

- Knows only its own data
- Has zero dependencies on other systems
- Doesn't know the game exists
- Can be reused in a completely different game

**The Dumb Test:** Can this system work in a completely different game without modification? If yes, it's properly Layer 0.

```gdscript
# turn_system.gd — Layer 0 (dumb)
class_name TurnSystem extends RefCounted

signal turn_changed(index: int)

var current_index: int = 0
var participant_count: int = 0


func setup(count: int) -> void:
    participant_count = count
    current_index = 0


func next_turn() -> int:
    current_index = (current_index + 1) % participant_count
    turn_changed.emit(current_index)
    return current_index


func get_current() -> int:
    return current_index
```

This system doesn't know:

- Who is fighting
- If someone died
- What the UI should show
- If combat is paused
- That "combat" even exists

It only knows: there's a count, there's an index, here's how to advance.

### Layer 1: Smart Orchestration

A Layer 1 manager:

- Connects Layer 0 systems together
- Contains game-specific logic
- Knows the context (combat, exploration, dialogue)
- Coordinates flow between systems

```gdscript
# combat_manager.gd — Layer 1 (smart)
class_name CombatManager extends Node

var turn_system: TurnSystem
var health_system: HealthSystem
var inventory_system: InventorySystem

var combatants: Array[CombatantInstance] = []

@onready var combat_ui: CombatUI = $CombatUI


func start_combat(participants: Array[CombatantInstance]) -> void:
    combatants = participants
    turn_system.setup(combatants.size())
    combat_ui.show()
    _process_turn()


func _process_turn() -> void:
    var current := turn_system.get_current()
    var unit := combatants[current]
    
    # Ask HealthSystem (Layer 0): is this unit dead?
    if health_system.is_dead(unit):
        turn_system.next_turn()
        _process_turn()
        return
    
    # Tell UI: show whose turn it is
    combat_ui.show_turn_indicator(unit)
    
    # Wait for player input or AI decision...


func _on_enemy_killed(enemy: CombatantInstance) -> void:
    # Orchestrate multiple Layer 0 systems
    var loot := loot_table.roll(enemy.definition)
    
    for item in loot:
        inventory_system.add(item)  # Layer 0 call
        quest_system.notify_event(&"item_acquired", item)  # Layer 0 call
    
    combat_ui.show_loot_popup(loot)  # View call
```

The CombatManager knows the game. The TurnSystem, HealthSystem, and InventorySystem don't.

### Micro-Managers: Technical Complexity Isolation

For complex technical tasks that would clutter gameplay code, create specialized systems:

```gdscript
# targeting_system.gd — Micro-manager for raycasting/targeting
class_name TargetingSystem extends RefCounted

var camera: Camera3D
var valid_targets: Array[Node3D] = []


func get_target_under_mouse() -> Node3D:
    var mouse_pos := camera.get_viewport().get_mouse_position()
    var from := camera.project_ray_origin(mouse_pos)
    var to := from + camera.project_ray_normal(mouse_pos) * 1000
    
    # Raycast logic...
    return _find_closest_valid_target(from, to)


func is_valid_target(node: Node3D) -> bool:
    return node in valid_targets
```

This keeps raycasting math out of your combat logic. The CombatManager just calls `targeting_system.get_target_under_mouse()`.

### Wishlist-First Design

Don't architect systems speculatively. Start with what the game needs:

```
Feature Wishlist:
- Turn-based combat
- Inventory with stacking
- Dialogue with choices
- Quest tracking
- Save/load
```

Then derive minimal systems to support those features:

```
Layer 0 Systems needed:
- TurnSystem (index tracking)
- InventorySystem (item storage)
- DialogueSystem (conversation state)
- QuestSystem (objective tracking)
- HealthSystem (HP management)

Layer 1 Managers needed:
- CombatManager (orchestrates combat flow)
- ExplorationManager (orchestrates world interaction)
- GameStateManager (orchestrates save/load, scene transitions)
```

No generic engine-building. No "we might need this later." Only what the wishlist demands.

### The Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                    LAYER 1: ORCHESTRATION                       │
│                    (Smart — knows the game)                     │
│                                                                 │
│  ┌─────────────┐  ┌──────────────┐  ┌───────────────────────┐   │
│  │CombatManager│  │ExploreManager│  │  GameStateManager     │   │
│  └──────┬──────┘  └──────┬───────┘  └───────────┬───────────┘   │
│         │                │                      │               │
└─────────┼────────────────┼──────────────────────┼───────────────┘
          │ calls          │ calls                │ calls
          ▼                ▼                      ▼
┌─────────────────────────────────────────────────────────────────┐
│                    LAYER 0: DATA & STATE                        │
│                    (Dumb — no game context)                     │
│                                                                 │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐            │
│  │   Turn   │ │  Health  │ │Inventory │ │  Quest   │   ...      │
│  │  System  │ │  System  │ │  System  │ │  System  │            │
│  └────┬─────┘ └────┬─────┘ └────┬─────┘ └────┬─────┘            │
│       │            │            │            │                  │
└───────┼────────────┼────────────┼────────────┼──────────────────┘
        │ owns       │ owns       │ owns       │ owns
        ▼            ▼            ▼            ▼
┌─────────────────────────────────────────────────────────────────┐
│                         DATA                                    │
│  ┌──────────────────┐  ┌──────────────────┐                     │
│  │    Definitions   │  │    Instances     │                     │
│  │  (static .tres)  │  │ (runtime state)  │                     │
│  └──────────────────┘  └──────────────────┘                     │
└─────────────────────────────────────────────────────────────────┘
```

### Where Game Logic Lives

|Question|Answer|
|---|---|
|"What happens when a turn ends?"|Layer 1 (CombatManager)|
|"How do I advance to the next turn?"|Layer 0 (TurnSystem)|
|"What happens when an enemy dies?"|Layer 1 (CombatManager)|
|"Is this unit dead?"|Layer 0 (HealthSystem)|
|"What happens when I pick up an item?"|Layer 1 (ExplorationManager)|
|"How do I add an item to inventory?"|Layer 0 (InventorySystem)|

**Rule of thumb:** Layer 0 answers "how do I do X?" — Layer 1 answers "what should happen when X?"

### Communication Rules

Layer 0 systems:

- Emit signals (blindly, not knowing who listens)
- Do NOT listen to other Layer 0 systems' signals
- Do NOT call other Layer 0 systems directly

If `HealthSystem` needs to affect `InventorySystem` when someone dies, that's **game logic** — it belongs in Layer 1.

|From|To|Method|
|---|---|---|
|View → Layer 0|Direct call|`inventory.add(item)`|
|View → Layer 1|Direct call|`combat_manager.end_turn()`|
|Layer 1 → Layer 0|Direct call|`health_system.take_damage(...)`|
|Layer 0 → Layer 1|Signal|`entity_died.emit(...)`|
|Layer 0 → View|Signal|`item_added.emit(...)`|
|Layer 0 → Layer 0|**NEVER**|❌ This is game logic, belongs in Layer 1|

**The key insight:** Layer 1 is the glue. When system A's event should affect system B, Layer 1 listens to A and calls B. The systems themselves stay dumb and decoupled.

### When Should Views Call Layer 0 vs Layer 1?

|Situation|Call|Example|
|---|---|---|
|Simple data operation|Layer 0 directly|`inventory.add(item)`|
|Single system, no side effects|Layer 0 directly|`health_system.get_current(entity)`|
|Multiple systems need coordination|Layer 1|Pickup → quest update → achievement → sound|
|Game-specific flow|Layer 1|`combat_manager.end_turn()`|

For simple cases (like the `world_item._on_picked_up()` example), calling Layer 0 directly is fine. When an action triggers cascading effects across multiple systems, route through Layer 1.

### Layer 1 to Layer 1 Communication

Managers can call each other directly when needed:

```gdscript
# combat_manager.gd
func _on_combat_ended(victory: bool) -> void:
    if victory:
        dialogue_manager.start_dialogue(&"victory_speech")
        exploration_manager.unlock_area(&"next_zone")
```

However, if managers are constantly cross-calling, consider:

1. **Missing Layer 0 system** — shared logic should move down
2. **Overly broad manager** — split into focused managers
3. **Event bus pattern** — for truly decoupled communication

### When to Split a Manager

A manager is too big when:

- It handles unrelated responsibilities (combat + inventory UI + saving)
- You can't describe its job in one sentence
- Multiple developers need to edit it simultaneously
- It's over ~500 lines with no clear sections

Split by **game context**, not by technical function:

|Too Broad|Better Split|
|---|---|
|`GameManager` (does everything)|`CombatManager`, `ExplorationManager`, `UIManager`|
|`PlayerManager` (movement + combat + inventory + quests)|`PlayerController` (movement), `CombatManager`, `InventoryManager`|

Each manager should own one "mode" or "context" of gameplay.

See also: [[Systems access approaches in Godot]]

---

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

```gdscript
# world_item_data.gd
class_name WorldItemData extends Resource

@export var item: ItemInstance
@export var position: Vector3
```

### Save Manager

```gdscript
# save_manager.gd
class_name SaveManager extends Node  # Autoload

const SAVE_PATH := "user://savegame.tres"

@onready var inventory: InventorySystem = GameSystems.inventory
@onready var world_state: WorldStateSystem = GameSystems.world


func save_game() -> void:
    var save := SaveGame.new()
    save.player_position = player.global_position
    save.player_health = player.health
    save.inventory_items = inventory.get_all_items()
    save.world_items = _collect_world_items()
    save.play_time_seconds = Time.get_ticks_msec() / 1000.0
    
    var error := ResourceSaver.save(save, SAVE_PATH)
    if error != OK:
        push_error("Failed to save: %s" % error)


func load_game() -> void:
    if not ResourceLoader.exists(SAVE_PATH):
        push_warning("No save file found")
        return
    
    # CACHE_MODE_IGNORE ensures fresh copy, not cached resource
    var save: SaveGame = ResourceLoader.load(
        SAVE_PATH, "", ResourceLoader.CACHE_MODE_IGNORE
    )
    
    player.global_position = save.player_position
    player.health = save.player_health
    inventory.set_all_items(save.inventory_items)
    _spawn_world_items(save.world_items)


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
        get_tree().current_scene.add_child(node)
```

### How Definition References Survive Save/Load

When Godot saves an `ItemInstance`, it stores definition as a resource path:

```
[resource]
definition = ExtResource("res://data/items/iron_axe.tres")
durability = 47
quantity = 1
```

On load, Godot resolves the path and assigns the definition. Instances automatically reconnect to shared definitions.

### Save: No Duplication Needed

```gdscript
save.inventory_items = inventory.items  # Direct reference is fine
ResourceSaver.save(save, SAVE_PATH)     # Serializes immediately
```

### Load: Use CACHE_MODE_IGNORE

```gdscript
# Without CACHE_MODE_IGNORE, you get cached resource
# Modifying it modifies the cache, corrupting future loads
var save: SaveGame = ResourceLoader.load(
    SAVE_PATH, "", ResourceLoader.CACHE_MODE_IGNORE
)
```

---

## Manual Serialization Alternative

For binary format, encryption, or JSON export:

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
func save_game_binary() -> void:
    var data := {
        "version": 1,
        "inventory": inventory.get_all_items().map(func(i): return i.to_dict()),
    }
    var file := FileAccess.open("user://save.dat", FileAccess.WRITE)
    file.store_var(data)


func load_game_binary() -> void:
    var file := FileAccess.open("user://save.dat", FileAccess.READ)
    var data: Dictionary = file.get_var()
    
    for item_data in data["inventory"]:
        var inst := ItemInstance.from_dict(item_data, ItemDatabase)
        inventory.add(inst)
```

---

## Common Patterns

### Spawning Items

```gdscript
# From definition ID
func spawn_item(id: StringName, pos: Vector3) -> void:
    var instance := ItemDatabase.create_instance(id)
    var node := preload("res://scenes/world_item.tscn").instantiate()
    node.setup(instance)
    node.global_position = pos
    world.add_child(node)

# From existing instance (dropping from inventory)
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

---

## Extending the Pattern

### Inheritance for Item Types

```gdscript
class_name WeaponDefinition extends ItemDefinition

@export var damage: int
@export var attack_speed: float
```

```gdscript
class_name ConsumableDefinition extends ItemDefinition

@export var heal_amount: int
@export var buff_duration: float
```

### Instance-Specific State

```gdscript
class_name WeaponInstance extends ItemInstance

@export var enchantments: Array[Enchantment] = []
@export var bonus_damage: int = 0

var total_damage: int:
    get: 
        var base: int = (definition as WeaponDefinition).damage
        return base + bonus_damage
```

---

## Data Separation Levels

Not every feature needs full Definition + Instance separation.

### Definition-Only

Static template, mutable state lives on the node itself.

```gdscript
# EnemyDefinition resource
@export var max_health := 100

# Enemy node
@export var definition: EnemyDefinition
var health: int

func _ready():
    health = definition.max_health
```

Use when: data should be editable/shareable, but node is source of truth at runtime.

### Definition + Instance

Separate resources for static template and mutable state.

Use when: data's lifecycle is independent of any node (inventory, saves, transfers).

|Question|Answer|
|---|---|
|Want data out of code?|Definition-only minimum|
|Does mutable state outlive the node?|Need Instance layer|

---

## Quick Reference

|Want to...|Do this|
|---|---|
|Create new item type|New `Resource` script extending `ItemDefinition`|
|Add runtime state|Add `@export var` to `ItemInstance`|
|Spawn item in world|`ItemDatabase.create_instance(id)` → node.setup|
|Pick up item|Pass instance to inventory, `queue_free()` node|
|Save game|`ResourceSaver.save(save_game, path)`|
|Load game|`ResourceLoader.load(path, "", CACHE_MODE_IGNORE)`|
|Look up definition|`ItemDatabase.get_definition(id)`|
|Add data operation|Layer 0 system method (e.g., `inventory.add()`)|
|Add game-specific logic|Layer 1 manager (e.g., `CombatManager._on_kill()`)|
|Isolate technical code|Micro-manager (e.g., `TargetingSystem`)|

---

## Checklist

- [ ] Definitions are `Resource` scripts with `@export` vars
- [ ] Definitions are `.tres` files in `res://data/`
- [ ] Instances hold reference to definition + mutable state
- [ ] Instances also extend `Resource` (for saving)
- [ ] Nodes hold instance reference, not definition
- [ ] Database uses `ResourceLoader.list_directory()` or explicit array (not `DirAccess`)
- [ ] Layer 0 systems are dumb (pass the "different game" test)
- [ ] Layer 0 systems never call or listen to other Layer 0 systems
- [ ] Layer 1 managers orchestrate systems and contain game logic
- [ ] SaveGame resource wraps all saveable state
- [ ] Load with `CACHE_MODE_IGNORE`

---

## Gotchas

1. **Don't modify definitions at runtime** — changes affect all instances and persist in editor
2. **`duplicate()` is shallow** — nested Arrays/Dictionaries need `duplicate(true)` or explicit recreation
3. **Resource caching** — use `CACHE_MODE_IGNORE` on load to avoid corrupted state
4. **Export vars required** — only `@export` properties are serialized by `ResourceSaver`
5. **DirAccess breaks on export** — use `ResourceLoader.list_directory()` for scanning resource folders
6. **System lifecycle** — RefCounted systems get garbage collected if nothing holds a reference

---

## Adapting for Small Projects

The full pattern (Definition + Instance + System + Manager + Database) is overkill for jams and small games. But the core principles still pay off—even a 48-hour jam benefits from not having spaghetti.

### What to Keep

**Always separate data from nodes.** This is the minimum viable version:

```gdscript
# enemy_definition.gd
class_name EnemyDefinition extends Resource

@export var id: StringName
@export var display_name: String
@export var max_health: int = 100
@export var damage: int = 10
@export var speed: float = 100.0
@export var sprite: Texture2D
```

```gdscript
# enemy.gd
extends CharacterBody2D

@export var definition: EnemyDefinition

var health: int

func _ready() -> void:
    health = definition.max_health
    $Sprite2D.texture = definition.sprite

func take_damage(amount: int) -> void:
    health -= amount
    if health <= 0:
        queue_free()
```

That's it. No Instance class, no System, no Database. The node holds mutable state directly. You still get:

- Reusable enemy types (drag `goblin.tres` or `skeleton.tres` onto any enemy node)
- Designer-friendly editing (no code changes to tweak stats)
- Type safety (can't accidentally assign a weapon to an enemy slot)

### What to Skip

|Full Pattern|Small Project Alternative|
|---|---|
|Instance class|Mutable state lives on the node|
|System (RefCounted)|Logic lives on the node or a simple Autoload|
|Manager (Node wrapper)|Not needed if no System|
|Database singleton|Direct `preload()` or `@export` references|
|SaveGame resource|`ConfigFile` or simple `store_var()`|

### Lightweight Database

If you have 10+ definitions and want lookup by ID without the full Database pattern:

```gdscript
# game_data.gd (Autoload)
extends Node

var enemies: Dictionary = {}

func _ready() -> void:
    _register("res://data/enemies/goblin.tres")
    _register("res://data/enemies/skeleton.tres")
    _register("res://data/enemies/boss.tres")

func _register(path: String) -> void:
    var def: EnemyDefinition = load(path)
    enemies[def.id] = def

func get_enemy(id: StringName) -> EnemyDefinition:
    return enemies.get(id)
```

Explicit paths, no scanning, no complexity. Works fine for small games.

### Lightweight Save System

Skip the SaveGame resource pattern. Use `ConfigFile` for simple key-value data:

```gdscript
# save_manager.gd (Autoload)
extends Node

const SAVE_PATH := "user://save.cfg"

func save_game() -> void:
    var config := ConfigFile.new()
    config.set_value("player", "position", player.global_position)
    config.set_value("player", "health", player.health)
    config.set_value("game", "level", current_level)
    config.set_value("game", "score", score)
    config.save(SAVE_PATH)

func load_game() -> void:
    var config := ConfigFile.new()
    if config.load(SAVE_PATH) != OK:
        return
    player.global_position = config.get_value("player", "position", Vector2.ZERO)
    player.health = config.get_value("player", "health", 100)
    current_level = config.get_value("game", "level", 1)
    score = config.get_value("game", "score", 0)
```

Or `store_var()` for more complex data:

```gdscript
func save_game() -> void:
    var data := {
        "position": player.global_position,
        "health": player.health,
        "inventory": player.inventory_ids,  # Array of StringNames
    }
    var file := FileAccess.open("user://save.dat", FileAccess.WRITE)
    file.store_var(data)

func load_game() -> void:
    var file := FileAccess.open("user://save.dat", FileAccess.READ)
    if not file:
        return
    var data: Dictionary = file.get_var()
    player.global_position = data.get("position", Vector2.ZERO)
    player.health = data.get("health", 100)
```

### When to Upgrade

Start simple. Add complexity when you feel the pain:

|Pain Point|Solution|
|---|---|
|Duplicating stat changes across enemy nodes|You need Definitions (Resource)|
|Node destruction loses data you need|You need Instances (separate Resource for runtime state)|
|Game logic scattered across 10 nodes|You need a System (centralized logic)|
|"Where does this get instantiated?" confusion|You need a Manager (explicit ownership)|
|Typos in string IDs causing bugs|You need a Database (lookup with validation)|

### Jam-Friendly Structure

For a 48-hour jam, this structure is usually enough:

```
res://
├── data/
│   ├── enemies/
│   │   ├── goblin.tres
│   │   └── skeleton.tres
│   └── items/
│       ├── sword.tres
│       └── potion.tres
├── scenes/
│   ├── enemy.tscn      # has @export var definition: EnemyDefinition
│   └── item.tscn       # has @export var definition: ItemDefinition
├── scripts/
│   ├── enemy_definition.gd
│   ├── item_definition.gd
│   └── game_manager.gd  # Autoload with simple save/load + game state
└── project.godot
```

No Instances, no Systems, no Managers. Just Definitions + Nodes + one Autoload for global state. You can always refactor later—starting with separated data makes that refactor painless.

### The Minimum Principle

> **Separate data from presentation. Everything else is optional.**

Even the simplest separation—a Resource with `@export` vars instead of hardcoded values in the node—makes your code:

- Easier to tweak (edit `.tres` in inspector, not code)
- Easier to reuse (same node, different definition)
- Easier to extend (add fields to Resource, not refactor nodes)

The full pattern exists for when you need it. Start with Definitions, add layers only when the current approach hurts.