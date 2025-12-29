# Data-Driven Design in Godot 4

A reference for separating data from presentation, handling mutable state, and implementing save systems.

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
│                   SYSTEMS & MANAGERS                            │
│  Operate on data. Managers coordinate, Systems are shared ops.  │
│                                                                 │
│  - CombatManager.start_combat() — game-specific coordination    │
│  - HealthSystem.damage() — shared operation                     │
│  - InventorySystem.add() — shared operation                     │
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

> **Note:** Systems only exist for shared operations (used by multiple managers). Managers can contain data operations if they're not shared. See [[#Systems vs Managers]] for details.

### Why This Order?

1. **Data first** — Forces you to understand the problem before coding
2. **Logic second** — Managers coordinate, systems handle shared operations
3. **Views last** — Just render current state, trivially replaceable

### System Ownership

Systems extend `RefCounted` and get garbage collected if nothing holds a reference. They must be either:

- **Owned by GameSystems autoload** — central container for shared systems
- **Owned by a Manager** — if private to that manager

---

## The Two Types of Data

|Type|Purpose|Mutability|Storage|
|---|---|---|---|
|**Definition**|Template describing _what_ something is|Read-only|`.tres` files in `res://`|
|**Instance**|Specific occurrence with runtime state|Mutable|Created at runtime, saved to `user://`|

### Why Separate Them?

- **Memory**: 100 iron axes share one `IronAxeDefinition`, not 100 copies
- **Saving**: Only serialize what changed (durability: 47), not the entire definition
- **Editing**: Change `IronAxeDefinition.damage` once, all axes update
- **Clarity**: Obvious what's static vs dynamic

### When Instance is Overkill

**Don't create an Instance class when the node IS the runtime state:**

```gdscript
# Enemy that dies and is gone — no Instance needed
extends CharacterBody3D

@export var definition: EnemyDefinition
var health: int

func _ready() -> void:
    health = definition.max_health

func take_damage(amount: int) -> void:
    health -= amount
    if health <= 0:
        queue_free()  # Node dies, data dies with it
```

The node holds mutable state. When it's destroyed, the data is gone. No save, no transfer.

**Create an Instance class when:**

|Situation|Why|
|---|---|
|Data survives node destruction|Inventory items persist after pickup|
|Data transfers between scenes|Player stats carry across levels|
|Data needs to be saved/loaded|Save file stores durability, enchantments|
|Multiple views display same data|Inventory grid + tooltip show same item|

**Rule:** Start with Definition-only. Add Instance when data needs to outlive its node.

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

### When Database is Overkill

**Don't create a Database when you know exactly what you need at edit time:**

```gdscript
# Direct references — no database needed
@export var goblin: EnemyDefinition    # Drag in inspector
@export var skeleton: EnemyDefinition
@export var boss: EnemyDefinition

func spawn_wave_1() -> void:
    spawn(goblin)
    spawn(goblin)
    spawn(skeleton)
```

This works fine for small games. You know at edit time which enemies appear where.

**Create a Database when:**

|Situation|Why|
|---|---|
|Spawning from string IDs|Data files, network messages, console commands|
|10+ definitions and growing|Lookup is easier than managing many @export vars|
|Multiple systems need same lookup|Avoid passing references everywhere|
|Dynamic content|Mods, procedural generation|

**Rule:** Start with direct `@export` references or `preload()`. Add Database when lookup-by-ID becomes necessary.

---

## Systems vs Managers

This architecture draws from Ricard Pillosu's approach (20+ years AAA, currently building Starfinder in Godot).
Source: [Jonas Tyroller Podcast — "Industry Veteran About Coding for Games"](https://www.youtube.com/watch?v=Fxea6TG0PXk) (1:02:43 - 1:20:48)

### The Core Distinction

| System                 | Manager                                |                                 |
| ---------------------- | -------------------------------------- | ------------------------------- |
| **Purpose**            | Shared operations on data              | Game-specific coordination      |
| **Knows game context** | No                                     | Yes                             |
| **When to create**     | Multiple managers need same operations | Event triggers multiple effects |
| **Extends**            | RefCounted                             | Node                            |

**The key insight:** A system only exists because multiple parts of your game need the same operations. If nothing shares it, keep the code in the manager.

### When to Extract a System

**Extract when multiple managers need the same operations:**

```gdscript
# health_system.gd — extracted because shared
class_name HealthSystem extends RefCounted

# Used by: CombatManager, PlayerManager, TrapManager, PoisonManager
func damage(entity, amount: int) -> void:
    entity.health -= amount
    if entity.health <= 0:
        entity_died.emit(entity)
```

**Don't extract when only one manager uses it:**

```gdscript
# day_cycle_manager.gd — no DayCycleSystem needed
class_name DayCycleManager extends Node

var current_phase: DayPhaseDefinition
var phase_timer: float = 0.0

func _process(delta: float) -> void:
    phase_timer += delta
    if phase_timer >= current_phase.duration:
        _advance_phase()

func _advance_phase() -> void:
    # This is data manipulation, but only DayCycleManager uses it
    # No reason to extract to a separate system
    current_phase_index = (current_phase_index + 1) % phases.size()
    phase_timer = 0.0
    
    # Coordination part
    GameSystems.spawning.set_multiplier(current_phase.spawn_rate)
    MusicManager.crossfade(current_phase.music)
    phase_changed.emit(current_phase)
```

DayCycleManager owns data AND operates on it. That's fine — it's the only thing that touches day/night logic.

### When to Create a Manager

**Create when an event triggers multiple effects:**

```gdscript
# combat_manager.gd
func _on_enemy_killed(enemy: CombatantInstance) -> void:
    # Multiple systems need coordination
    var loot := loot_table.roll(enemy.definition)
    inventory_system.add(loot)
    quest_system.notify(&"enemy_killed", enemy)
    audio_system.play(&"death_sound")
    combat_ui.show_loot_popup(loot)
```

**Don't create when the action is one call:**

```gdscript
# Bad — SafeZoneManager just forwards a call
func _on_body_entered(body):
    SafeZoneManager.enter()  # which just calls PlayerManager.set_safe(true)

# Good — direct call, no middleman
func _on_body_entered(body):
    PlayerManager.set_safe(true)
```

### The Decision Flow

```
Is this operation used by multiple managers?
├── Yes → Extract to a System
└── No  → Keep in the Manager

Does this event trigger multiple effects?
├── Yes → Create a Manager to coordinate
└── No  → View can call System/Manager directly
```

### A Manager Can Own Data

Managers aren't just coordinators — they can own data and operate on it:

```gdscript
# player_manager.gd
class_name PlayerManager extends Node

# Owns data
var is_safe: bool = false
var cold_damage_rate: float = 5.0

# Operates on its own data
func _process(delta: float) -> void:
    if not is_safe:
        GameSystems.health.damage(player, cold_damage_rate * delta)

# Coordinates
func set_safe(safe: bool) -> void:
    is_safe = safe
    if safe:
        GameSystems.health.stop_dot(player, &"cold")
    safety_changed.emit(safe)
```

Only extract to a system if another manager needs the same operations.

### System Example (Shared Operations)

```gdscript
# health_system.gd
class_name HealthSystem extends RefCounted

signal health_changed(entity, new_health: int)
signal entity_died(entity)

func damage(entity, amount: int) -> void:
    entity.health -= amount
    health_changed.emit(entity, entity.health)
    if entity.health <= 0:
        entity_died.emit(entity)

func heal(entity, amount: int) -> void:
    entity.health = min(entity.health + amount, entity.max_health)
    health_changed.emit(entity, entity.health)

func is_dead(entity) -> bool:
    return entity.health <= 0
```

This exists because CombatManager, PlayerManager, TrapManager all need `damage()`, `heal()`, `is_dead()`.

### Manager Example (Game Coordination)

```gdscript
# combat_manager.gd
class_name CombatManager extends Node

var turn_system: TurnSystem
var health_system: HealthSystem

func start_combat(participants: Array[CombatantInstance]) -> void:
    turn_system.setup(participants.size())
    combat_ui.show()
    _process_turn()

func _process_turn() -> void:
    var unit := combatants[turn_system.get_current()]
    
    if health_system.is_dead(unit):
        turn_system.next_turn()
        _process_turn()
        return
    
    combat_ui.show_turn_indicator(unit)
```

CombatManager knows the game — it knows what "combat" means, what should happen each turn, how death affects flow.

### Micro-Systems: Technical Complexity Isolation

For complex technical tasks that would clutter gameplay code:

```gdscript
# targeting_system.gd
class_name TargetingSystem extends RefCounted

var camera: Camera3D

func get_target_under_mouse() -> Node3D:
    var mouse_pos := camera.get_viewport().get_mouse_position()
    var from := camera.project_ray_origin(mouse_pos)
    var to := from + camera.project_ray_normal(mouse_pos) * 1000
    # Raycast logic...
    return _find_closest_valid_target(from, to)
```

Keeps raycasting math out of CombatManager. Not shared, but isolated for clarity.

### Wishlist-First Design

Don't architect speculatively. Start with what the game needs:

```
Feature Wishlist:
- Turn-based combat
- Inventory with stacking
- Quest tracking
- Day/night cycle
```

Then ask: **What operations are shared?**

```
Shared (extract to System):
- Health operations (damage, heal) — used by combat, traps, environment
- Inventory operations (add, remove) — used by loot, shops, crafting
- Turn operations — used by combat, maybe dialogue

Not shared (keep in Manager):
- Day/night progression — only DayCycleManager touches it
- Combat flow logic — only CombatManager
- Quest state changes — only QuestManager
```

### When NOT to Create a System

|Situation|Do This Instead|
|---|---|
|Only one manager uses this operation|Keep it in the manager|
|"Might need it later"|Wait until you actually need it|
|Feels cleaner to separate|Not a good enough reason|
|It's only 10 lines of code|Keep it in the manager|

### When NOT to Create a Manager

|Situation|Do This Instead|
|---|---|
|Action is just one call|View calls System directly|
|Manager just forwards calls|Remove the middleman|
|"Every feature needs a manager"|No, only coordination needs managers|
|No shared state or multi-step logic|Keep it simple|

### The Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                       MANAGERS                                  │
│            (Smart — know the game, coordinate)                  │
│                                                                 │
│  ┌─────────────┐  ┌──────────────┐  ┌───────────────────────┐   │
│  │CombatManager│  │PlayerManager │  │  DayCycleManager      │   │
│  └──────┬──────┘  └──────┬───────┘  └───────────┬───────────┘   │
│         │                │                      │               │
└─────────┼────────────────┼──────────────────────┼───────────────┘
          │ calls          │ calls                │ may own data
          ▼                ▼                      │ if not shared
┌─────────────────────────────────────────────────────────────────┐
│                  SYSTEMS (Shared Operations)                    │
│              (Dumb — reusable, no game context)                 │
│                                                                 │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐                         │
│  │  Health  │ │Inventory │ │   Turn   │                         │
│  │  System  │ │  System  │ │  System  │   (only if shared)      │
│  └────┬─────┘ └────┬─────┘ └────┬─────┘                         │
│       │            │            │                               │
└───────┼────────────┼────────────┼───────────────────────────────┘
        │ owns       │ owns       │ owns
        ▼            ▼            ▼
┌─────────────────────────────────────────────────────────────────┐
│                         DATA                                    │
│  ┌──────────────────┐  ┌──────────────────┐                     │
│  │    Definitions   │  │    Instances     │                     │
│  │  (static .tres)  │  │ (runtime state)  │                     │
│  └──────────────────┘  └──────────────────┘                     │
└─────────────────────────────────────────────────────────────────┘
```

Note: Systems only exist if multiple managers need the same operations. Otherwise, keep the logic in the manager.

### Where Logic Lives

|Question|Answer|
|---|---|
|"What happens when a turn ends?"|Manager (CombatManager) — game rule|
|"How do I advance to the next turn?"|System (TurnSystem) — shared operation|
|"What happens when an enemy dies?"|Manager (CombatManager) — game rule|
|"Is this unit dead?"|System (HealthSystem) — shared operation|
|"What happens when I enter safe zone?"|Manager (PlayerManager) — game rule|
|"How do I damage an entity?"|System (HealthSystem) — shared operation|

**Rule of thumb:** Managers answer "what should happen?" — Systems answer "how do I do it?"

### Communication Rules

Systems:

- Emit signals (blindly, not knowing who listens)
- Do NOT call other systems directly
- If system A needs to affect system B, a manager listens to A and calls B

|From|To|Method|
|---|---|---|
|View → System|Direct call|`GameSystems.inventory.add(item)`|
|View → Manager|Direct call|`CombatManager.end_turn()`|
|Manager → System|Direct call|`health_system.damage(...)`|
|System → Manager|Signal|`entity_died.emit(...)`|
|System → View|Signal|`item_added.emit(...)`|
|System → System|**NEVER**|❌ Manager coordinates this|

### When Should Views Call System vs Manager?

|Situation|Call|Example|
|---|---|---|
|Simple data operation|System directly|`GameSystems.inventory.add(item)`|
|Single operation, no side effects|System directly|`GameSystems.health.get_current(entity)`|
|Multiple effects needed|Manager|Pickup → quest + achievement + sound|
|Game-specific flow|Manager|`CombatManager.end_turn()`|

For simple cases, calling System directly is fine. When an action triggers multiple effects, route through a Manager.

### Manager to Manager Communication

Managers can call each other directly:

```gdscript
# combat_manager.gd
func _on_combat_ended(victory: bool) -> void:
    if victory:
        DialogueManager.start(&"victory_speech")
        ExplorationManager.unlock(&"next_zone")
```

If managers constantly cross-call, consider:

1. **Missing shared system** — extract common operations
2. **Overly broad manager** — split into focused managers
3. **Event bus** — for truly decoupled communication

### When to Split a Manager

A manager is too big when:

- It handles unrelated responsibilities
- You can't describe its job in one sentence
- It's over ~500 lines with no clear sections

Split by **game context**, not by technical function:

|Too Broad|Better Split|
|---|---|
|`GameManager` (does everything)|`CombatManager`, `ExplorationManager`, `UIManager`|
|`PlayerManager` (movement + combat + inventory)|`PlayerController` (movement), `PlayerManager` (state)|

Each manager should own one "mode" or "context" of gameplay.

---

## Accessing Systems and Managers

Where do systems and managers live? How do you access them?

### Recommended: Systems Container + Manager Autoloads

```
GameSystems (Autoload) — holds shared systems
├── inventory: InventorySystem
├── health: HealthSystem
├── turn: TurnSystem
└── quest: QuestSystem

CombatManager (Autoload)
ExplorationManager (Autoload)  
SaveManager (Autoload)
```

```gdscript
# game_systems.gd (Autoload)
extends Node

var inventory: InventorySystem
var health: HealthSystem
var turn: TurnSystem
var quest: QuestSystem


func _ready() -> void:
    inventory = InventorySystem.new()
    health = HealthSystem.new()
    turn = TurnSystem.new()
    quest = QuestSystem.new()
```

```gdscript
# combat_manager.gd (Autoload)
extends Node

@onready var health: HealthSystem = GameSystems.health
@onready var turn: TurnSystem = GameSystems.turn
@onready var inventory: InventorySystem = GameSystems.inventory


func _ready() -> void:
    health.entity_died.connect(_on_entity_died)


func _on_entity_died(entity) -> void:
    if entity.is_enemy:
        var loot := _roll_loot(entity)
        inventory.add(loot)
```

**Access from anywhere:**

```gdscript
GameSystems.inventory.add(item)        # Shared system
CombatManager.start_combat(enemies)    # Manager
```

### Why This Structure?

|Benefit|Explanation|
|---|---|
|**Single source for systems**|All shared systems in one place: `GameSystems.x`|
|**Clear ownership**|Systems don't float around — `GameSystems` owns them|
|**No duplication**|Multiple managers share the same system instances|
|**Easy to find**|Managers are named autoloads: `CombatManager`, `SaveManager`|

### When GameSystems Container is Overkill

For small games with 1-2 shared systems, just make them autoloads directly:

```
# Project Settings → Autoload
InventorySystem → res://systems/inventory_system.gd
CombatManager → res://managers/combat_manager.gd
```

```gdscript
# Access directly
InventorySystem.add(item)
```

**Use GameSystems container when:**

- 3+ shared systems
- You want to initialize them in a specific order
- You want one place to find all systems

### Alternative: Managers Own Their Systems

Each manager instantiates the systems it needs:

```gdscript
# combat_manager.gd
extends Node

var turn_system: TurnSystem
var health_system: HealthSystem


func _ready() -> void:
    turn_system = TurnSystem.new()
    health_system = HealthSystem.new()
```

**Problem:** If both `CombatManager` and `ExplorationManager` need `HealthSystem`, who owns it? You get awkward cross-references or duplicated instances.

**Use only when:** A system is truly private to one manager and never shared.

### Alternative: Scene-Based Managers

Managers as nodes in the scene tree, not autoloads:

```
Main.tscn
├── World
├── Player
├── CombatManager (Node)
└── DialogueManager (Node)
```

```gdscript
# Access via group
var combat := get_tree().get_first_node_in_group("combat_manager") as CombatManager

# Or via explicit path/export
@export var combat_manager: CombatManager
```

**Use when:**

- Manager is scene-specific (puzzle manager only in puzzle levels)
- Manager needs scene tree access (`$Path`, `get_tree()`)
- Different scenes need different manager configurations

### Summary: Where Things Live

|Component|Where|Access Pattern|
|---|---|---|
|Shared Systems|`GameSystems` Autoload|`GameSystems.inventory`|
|Global Managers (Save, Audio)|Individual Autoloads|`SaveManager.save_game()`|
|Context Managers (Combat, Dialogue)|Autoloads or scene nodes|`CombatManager.x` or group/export|
|Database|Autoload|`ItemDatabase.get_definition(id)`|

### Project Setup

In `Project → Project Settings → Autoload`:

|Name|Path|
|---|---|
|GameSystems|`res://systems/game_systems.gd`|
|ItemDatabase|`res://systems/item_database.gd`|
|SaveManager|`res://managers/save_manager.gd`|
|CombatManager|`res://managers/combat_manager.gd`|

Folder structure:

```
res://
├── systems/
│   ├── game_systems.gd      # Autoload — holds shared systems
│   ├── inventory_system.gd  # Shared system
│   ├── health_system.gd     # Shared system
│   └── turn_system.gd       # Shared system
├── managers/
│   ├── combat_manager.gd    # Autoload
│   ├── save_manager.gd      # Autoload
│   └── exploration_manager.gd
├── data/
│   └── items/
│       └── *.tres
└── scenes/
    └── *.tscn
```

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
|Add shared operation|System (e.g., `GameSystems.health.damage()`)|
|Add game-specific logic|Manager (e.g., `CombatManager._on_enemy_killed()`)|
|Isolate technical code|Micro-system (e.g., `TargetingSystem`)|
|Access a system|`GameSystems.inventory`, `GameSystems.health`|
|Access a manager|`CombatManager.x`, `SaveManager.x` (Autoloads)|

---

## Checklist

- [ ] Definitions are `Resource` scripts with `@export` vars
- [ ] Definitions are `.tres` files in `res://data/`
- [ ] Instances hold reference to definition + mutable state (if needed)
- [ ] Instances also extend `Resource` (for saving)
- [ ] Database exists only if lookup-by-ID is needed
- [ ] Database uses `ResourceLoader.list_directory()` or explicit array (not `DirAccess`)
- [ ] Systems only exist for shared operations (multiple managers use them)
- [ ] Systems never call other systems (managers coordinate)
- [ ] Managers coordinate game-specific flows
- [ ] Managers can own data if not shared
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