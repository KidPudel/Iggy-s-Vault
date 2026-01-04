# Layered Architecture in Godot

> [!quote] The Core Insight Separate what IS true (simulation) from what SHOWS the truth (presentation). Your game should work without graphics.

---

## The Problem This Solves

Without architecture, code becomes a web:

```
Player.gd knows about:
  → Enemy.gd (to deal damage)
  → Inventory.gd (to check items)
  → QuestManager.gd (to notify kills)
  → AudioManager.gd (to play sounds)
  → UI.gd (to update health bar)
  → Camera.gd (to shake on hit)
  
Enemy.gd knows about:
  → Player.gd (to deal damage)
  → LootTable.gd (to drop items)
  → ... (grows forever)
```

Everything depends on everything. Change one thing, break ten others. Can't test anything in isolation. Can't reason about flow.

**The fix:** Layers with clear responsibilities and communication rules.

---

## The Three-Layer Model

This architecture draws from Ricard Pillosu's approach (20+ years AAA, currently building Starfinder in Godot).

```
┌─────────────────────────────────────────────────────────────────┐
│                    LAYER 0: THE TRUTH                           │
│                      (Systems)                                  │
│                                                                 │
│   "How do I do this operation on ANY entity?"                   │
│                                                                 │
│   - Pure data and logic                                         │
│   - Zero dependencies on visuals, UI, sound, scene tree         │
│   - Shared across multiple managers                             │
│   - Trivially unit testable (no scene required)                 │
│                                                                 │
│   Example: HealthSystem                                         │
│   - Knows current health of all entities                        │
│   - Knows how to damage/heal                                    │
│   - Emits "died" signal                                         │
│   - Knows NOTHING about death animations or loot drops          │
└─────────────────────────────────────────────────────────────────┘
                              │
                              │ Managers CALL systems
                              │ Systems EMIT signals
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                   LAYER 1: THE ORCHESTRATOR                     │
│                        (Managers)                               │
│                                                                 │
│   "What happens in THIS game when X occurs?"                    │
│                                                                 │
│   - Game-specific choreography                                  │
│   - Connects truth to presentation                              │
│   - Public API reads like a checklist                           │
│   - Private methods can contain logic (if single-use)           │
│                                                                 │
│   Example: CombatManager                                        │
│   - Hears "entity died" from HealthSystem                       │
│   - Removes from turn order                                     │
│   - Rolls and spawns loot                                       │
│   - Notifies quest system                                       │
│   - Checks if combat should end                                 │
└─────────────────────────────────────────────────────────────────┘
                              │
                              │ Managers TELL nodes what to do
                              │ Nodes CALL managers/systems for actions
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                   LAYER 2: PRESENTATION                         │
│                    (Scene Nodes)                                │
│                                                                 │
│   "I know what X means for MY type"                             │
│                                                                 │
│   - Type-specific behavior (door.interact ≠ chest.interact)     │
│   - Owns its visuals (animations, particles, meshes)            │
│   - Calls systems/managers for shared operations                │
│   - Manager doesn't need to know node's internals               │
│                                                                 │
│   Example: DoorNode                                             │
│   - Knows what "interact" means for a door                      │
│   - Checks if player has key                                    │
│   - Plays its own open animation                                │
│   - Manager just says "interact", door figures out the rest     │
└─────────────────────────────────────────────────────────────────┘
```

---

## The Stage Play Mental Model

|Component|Stage Equivalent|Knows...|
|---|---|---|
|**Systems**|The Script|What IS true (facts, state, rules)|
|**Managers**|The Director|WHEN things happen, WHO does WHAT|
|**Nodes**|The Actors|HOW to perform their specific part|

The script doesn't know about lighting. The director reads the script and cues actors. Actors are experts on their character—they don't need the director to explain how to cry.

> [!important] The Key Test Can you run your game logic without any nodes?
> 
> If yes, Layer 0 is properly isolated. If no, you've mixed presentation into truth.

---

## Relationship to Data-Driven Design

This document covers **code flow**—how logic is organized and how components communicate.

The [[Data-Driven Design in Godot]] document covers **data structure**—how to separate static templates (Definitions) from runtime state (Instances).

**They solve different problems:**

|Problem|Solution|
|---|---|
|How do I structure my game's **content**?|Data-Driven Design|
|How do I structure my game's **code flow**?|Layered Architecture|

**They work together:**

- Data-Driven Design gives you clean **data** to operate on
- Layered Architecture gives you clean **code** to operate with
- Systems (Layer 0) typically operate on Instances
- Nodes (Layer 2) display Definitions and Instances

You can use one without the other. A jam game might use Definition resources but keep all logic on nodes. A simulation might have clean Systems/Managers but store data in plain dictionaries.

---

## Systems: The Truth (Layer 0)

Systems are pure logic. They answer: **"How do I do this operation on ANY entity?"**

### What Makes a System

|Characteristic|Why|
|---|---|
|Extends `RefCounted` (not Node)|No scene tree dependency|
|Zero visuals/audio/UI|Doesn't know game has graphics|
|Only communicates via signals|Decoupled from listeners|
|Used by MULTIPLE managers|That's why it's extracted|
|Trivially testable|Call method, check result|

### What Goes in a System

|Belongs|Doesn't Belong|
|---|---|
|Data structures|Visual updates|
|State calculations|Sound effects|
|Rule evaluation|Camera movement|
|Damage formulas|Animation triggers|
|Turn order logic|Particle effects|

### Example: Health System

```gdscript
class_name HealthSystem
extends RefCounted

signal damaged(entity_id: int, amount: int, remaining: int)
signal healed(entity_id: int, amount: int, remaining: int)
signal died(entity_id: int)

var _health: Dictionary = {}      # entity_id → current
var _max_health: Dictionary = {}  # entity_id → maximum

func register(entity_id: int, max_hp: int) -> void:
    _max_health[entity_id] = max_hp
    _health[entity_id] = max_hp

func damage(entity_id: int, amount: int) -> void:
    var old_hp: int = _health.get(entity_id, 0)
    var new_hp: int = max(0, old_hp - amount)
    _health[entity_id] = new_hp
    
    damaged.emit(entity_id, amount, new_hp)
    
    if new_hp <= 0 and old_hp > 0:
        died.emit(entity_id)

func is_alive(entity_id: int) -> bool:
    return _health.get(entity_id, 0) > 0
```

Notice: No `$Path`, no `get_tree()`, no animations, no sounds. Pure data, pure logic.

### System Ownership

Systems are `RefCounted`—they get garbage collected if nothing holds a reference. Store them in an autoload:

```gdscript
# game_systems.gd (Autoload)
extends Node

var health: HealthSystem
var turn: TurnSystem
var inventory: InventorySystem

func _ready() -> void:
    health = HealthSystem.new()
    turn = TurnSystem.new()
    inventory = InventorySystem.new()
```

Access anywhere: `GameSystems.health.damage(id, 10)`

---

## Managers: The Orchestrator (Layer 1)

Managers are choreographers. They answer: **"What happens in THIS game when X occurs?"**

### What Makes a Manager

|Characteristic|Why|
|---|---|
|Extends `Node`|Needs lifecycle, can own children|
|Asks systems for truth|"Whose turn is it?"|
|Tells presentation what to show|"Display goblin turn"|
|Public API reads like English|Easy to follow flow|
|Game-specific logic lives here|Unless it's shared|

### The "Reads Like English" Rule

This applies to **public methods and signal handlers**—the choreography. Private helpers can have loops and conditionals.

```gdscript
# PUBLIC — reads like a checklist
func _on_entity_died(entity_id: int) -> void:
    _remove_from_combat(entity_id)
    _roll_and_spawn_loot(entity_id)
    _notify_quest_system(entity_id)
    _check_combat_end()

# PRIVATE — can contain actual logic
func _roll_and_spawn_loot(entity_id: int) -> void:
    var loot_table: LootTable = _combatants[entity_id].loot_table
    for entry in loot_table.entries:
        if randf() <= entry.chance:
            var qty := randi_range(entry.min_quantity, entry.max_quantity)
            var item := ItemDatabase.create_instance(entry.item_id, qty)
            _spawn_loot_node(item, _get_entity_position(entity_id))
```

The **public surface** tells the story. The **private guts** do the work.

### How Managers Talk to Presentation

Four mechanisms, each for different relationships:

|Relationship|Mechanism|Example|
|---|---|---|
|Manager owns the node (child)|Direct reference|`$CombatUI.show()`|
|Global service (one instance)|Autoload|`AudioManager.play()`|
|Manager needs specific node|Stored reference|`camera.enter_combat_mode()`|
|Loose coupling (many listeners)|Signal|`combat_started.emit()`|

```gdscript
# combat_manager.gd
@onready var combat_ui: Control = $CombatUI  # Owned child
var camera: Camera3D                          # Stored reference

func _ready() -> void:
    camera = get_tree().get_first_node_in_group("main_camera")
    GameSystems.health.died.connect(_on_entity_died)

func start_combat(enemy_ids: Array[int]) -> void:
    _in_combat = true
    _setup_combatants(enemy_ids)
    GameSystems.turn.setup([PlayerManager.get_player_id()] + enemy_ids)
    
    combat_ui.show()                    # Owned child — direct
    camera.enter_combat_mode()          # Stored reference — direct
    AudioManager.play(&"combat_start")  # Autoload — global
    combat_started.emit()               # Signal — for anything else
```

---

## Nodes: The Actors (Layer 2)

Nodes are the presentation layer. They answer: **"I know what X means for MY type."**

### The "Nodes Handle Themselves" Principle

> [!important] Godot's Core Philosophy A node is the expert on itself. It knows how to do its job. You don't need a central brain that knows everything about every object.

This is the key to avoiding giant match statements and keeping managers clean.

### The Problem

```gdscript
# BAD — Manager knows how every type works
func interact(target: Node) -> void:
    match target.type:
        "door":
            if target.is_locked:
                if PlayerManager.has_key(target.key_id):
                    target.unlock()
                    target.open()
                else:
                    DialogueManager.show("door_locked")
            else:
                target.open()
        "chest":
            # 15 lines of chest logic
        "campfire":
            # 10 lines of campfire logic
        # ... grows forever, manager knows EVERYTHING
```

Add new type = edit manager. Manager becomes unmaintainable.

### The Solution

```gdscript
# GOOD — Manager just coordinates
func interact(target: Node) -> void:
    if target.has_method("interact"):
        target.interact()  # Node handles itself
```

```gdscript
# door.gd — Door knows what "interact" means for a door
func interact() -> void:
    if is_locked and not PlayerManager.has_key(key_id):
        DialogueManager.show(&"door_locked")
        return
    _open()

func _open() -> void:
    animation_player.play("open")
    collision_shape.disabled = true
```

```gdscript
# chest.gd — Chest knows what "interact" means for a chest
func interact() -> void:
    if is_open:
        return
    is_open = true
    animation_player.play("open")
    for item in contents:
        GameSystems.inventory.add(item)
```

```gdscript
# campfire.gd — Campfire knows what "interact" means for a campfire
func interact() -> void:
    if GameSystems.inventory.has(&"stick"):
        GameSystems.inventory.remove(&"stick")
        fuel_remaining += 30.0
        particles.emitting = true
```

**Add new type = add new script.** Manager never changes.

### Two Kinds of "Execution"

|Layer|What Executes|Example|
|---|---|---|
|**System**|View-agnostic data operations|`health -= damage`|
|**Node**|Type-specific behavior WITH visuals|Play animation, spawn particles|

The flow:

```
SYSTEM: "Entity 5's health is now 0" (pure data change)
    ↓ emits died signal
MANAGER: "Entity 5 died → remove from combat, roll loot, check quest"
    ↓ tells node
NODE: "I'm Entity 5, I'll play death animation and queue_free myself"
```

### What Nodes Do

|Responsibility|Example|
|---|---|
|Type-specific behavior|`door.interact()` opens door, `chest.interact()` gives loot|
|Own their visuals|Play animations, manage particles, update meshes|
|Call managers/systems|`GameSystems.inventory.add(item)`|
|React to signals|Update health bar when `damaged` emits|

### What Nodes DON'T Do

|Forbidden|Why|
|---|---|
|Mutate other entities' data|Go through systems|
|Know about other node types|Manager coordinates|
|Contain shared operations|That's what systems are for|

---

## Where Does Logic Actually Live?

This is the decision that trips people up. Here's the priority order:

```
1. Is this operation used by MULTIPLE managers?
   └── Yes → Extract to SYSTEM
   └── No  ↓

2. Is this logic MESSY and cluttering the manager?
   └── Yes → Extract to HELPER (private class for isolation)
   └── No  ↓

3. Keep it in the MANAGER (private method)
```

### Examples

|Logic|Used By|Where It Lives|
|---|---|---|
|`damage(entity, amount)`|Combat, Traps, Environment, Poison|System (HealthSystem)|
|`roll_loot(enemy_id)`|Only CombatManager|Manager private method|
|`get_target_under_mouse()`|Only CombatManager, but messy raycast math|Helper (TargetingHelper)|
|Day/night phase progression|Only DayCycleManager|Manager private method|

### The Helper Pattern

When logic is ugly but single-use, extract to a helper for readability:

```gdscript
# targeting_helper.gd — isolates messy raycast math
class_name TargetingHelper
extends RefCounted

var camera: Camera3D

func get_target_under_mouse() -> Node3D:
    var mouse_pos := camera.get_viewport().get_mouse_position()
    var from := camera.project_ray_origin(mouse_pos)
    var to := from + camera.project_ray_normal(mouse_pos) * 1000
    # ... raycast logic
```

```gdscript
# combat_manager.gd — stays clean
var _targeting: TargetingHelper

func _unhandled_input(event: InputEvent) -> void:
    var target := _targeting.get_target_under_mouse()
    if target:
        attack(player_id, target.entity_id)
```

Helper isn't a "System" (not shared). It's just isolation for readability.

---

## Communication Rules

This is critical. Get it wrong and spaghetti returns.

```
┌─────────────────────────────────────────────────────────────────┐
│                   COMMUNICATION MATRIX                          │
├─────────────────────┬───────────────────────────────────────────┤
│  FROM → TO          │ METHOD                                    │
├─────────────────────┼───────────────────────────────────────────┤
│  Node → System      │ Direct call                               │
│  Node → Manager     │ Direct call                               │
│  Manager → System   │ Direct call                               │
│  Manager → Node     │ Direct call (owned) OR signal             │
│  System → Manager   │ SIGNAL ONLY                               │
│  System → Node      │ SIGNAL ONLY                               │
│  System → System    │ NEVER — manager coordinates               │
└─────────────────────┴───────────────────────────────────────────┘
```

### Why Systems Never Call Systems

```gdscript
# BAD — Hidden dependency
class_name HealthSystem

func damage(entity_id: int, amount: int) -> void:
    _health[entity_id] -= amount
    if _health[entity_id] <= 0:
        died.emit(entity_id)
        TurnSystem.remove_participant(entity_id)  # WRONG!
```

Problems:

- HealthSystem now depends on TurnSystem
- Can't use HealthSystem without TurnSystem
- Hidden side effect, hard to trace

```gdscript
# GOOD — Manager coordinates
# health_system.gd
func damage(entity_id: int, amount: int) -> void:
    _health[entity_id] -= amount
    if _health[entity_id] <= 0:
        died.emit(entity_id)  # Just announces

# combat_manager.gd
func _on_entity_died(entity_id: int) -> void:
    GameSystems.turn.remove_participant(entity_id)  # Manager coordinates
    _show_death_effects(entity_id)
```

Now the coordination is visible in one place.

---

## When to Create What

### The Fundamental Question: Why Not Just Call Directly?

```gdscript
# Chest opens, gives items. Why involve a manager?
func interact() -> void:
    for item in contents:
        GameSystems.inventory.add(item)
    animation_player.play("open")
```

This is **perfectly fine**. The node calls the system directly. No manager needed.

> [!important] The Core Rule **Managers exist for coordination, not for single operations.**
> 
> If an action only affects one system, the node calls that system directly. If an action ripples across multiple unrelated systems, a manager coordinates.

### Decision Tree

```
START: You have an action/event in your game
                    │
                    ▼
┌─────────────────────────────────────────────────────────────────┐
│ Is this operation needed by MULTIPLE parts of the game?         │
│ (e.g., damage calculation used by combat, traps, environment)   │
└─────────────────────────────────────────────────────────────────┘
                    │
          ┌────────┴────────┐
         YES                NO
          │                  │
          ▼                  ▼
    ┌──────────┐    ┌─────────────────────────────────────────────┐
    │ SYSTEM   │    │ Does this action trigger effects in          │
    │          │    │ MULTIPLE unrelated systems?                  │
    └──────────┘    └─────────────────────────────────────────────┘
                             │
                   ┌────────┴────────┐
                  YES                NO
                   │                  │
                   ▼                  ▼
             ┌──────────┐    ┌─────────────────────────────────────┐
             │ MANAGER  │    │ Is the logic messy/technical and    │
             │          │    │ cluttering the node?                │
             └──────────┘    └─────────────────────────────────────┘
                                      │
                            ┌────────┴────────┐
                           YES                NO
                            │                  │
                            ▼                  ▼
                      ┌──────────┐    ┌──────────────────┐
                      │ HELPER   │    │ NODE calls       │
                      │ class    │    │ system directly  │
                      └──────────┘    └──────────────────┘
```

### Concrete Examples

|Action|What It Triggers|Where Logic Lives|
|---|---|---|
|Chest gives loot|Add items to inventory|**Node** → System directly|
|Button plays click|Play sound|**Node** → AudioManager directly|
|Door opens|Animation, disable collision|**Node** handles itself|
|Player enters safe zone|Set one flag|**Node** → PlayerManager directly|
|Enemy takes damage|Reduce health|**Node** → System directly|
|Enemy dies|Turn order, loot, quest, achievements, combat end|**Manager** coordinates|
|Pick up quest item|Inventory + quest update + maybe dialogue|**Manager** coordinates|
|Day becomes night|Lighting, spawns, NPCs, music|**Manager** coordinates|

### Two Patterns for Manager Involvement

**Pattern A: Node Calls Manager Explicitly**

Node knows coordination is needed, routes through manager.

```gdscript
# node
func on_killed() -> void:
    CombatManager.handle_enemy_death(entity_id)
```

**Pattern B: Node Calls System, Manager Reacts to Signal**

Node does the simple thing. Manager hears the system's signal.

```gdscript
# node — doesn't know manager exists
func take_lethal_damage() -> void:
    GameSystems.health.damage(entity_id, 9999)

# system emits died(entity_id) signal

# manager — connected to health.died
func _on_entity_died(entity_id: int) -> void:
    GameSystems.turn.remove(entity_id)
    _roll_loot(entity_id)
    QuestManager.notify_kill(entity_id)
    _check_combat_end()
```

**Pattern B is more decoupled.** The node doesn't know a manager exists—it just damages, the system announces death, and whoever cares can listen.

### When Managers Emerge Naturally

You often don't need a manager until complexity grows:

```gdscript
# Version 1: Simple — no manager needed
func die() -> void:
    queue_free()

# Version 2: Drops loot — still fine in node
func die() -> void:
    _spawn_loot()
    queue_free()

# Version 3: Now touching 5 systems — time for a manager
func die() -> void:
    GameSystems.turn.remove(entity_id)
    _spawn_loot()
    QuestManager.notify(&"enemy_killed", definition.id)
    AchievementManager.increment(&"kills")
    if _is_boss():
        DialogueManager.trigger(&"boss_defeated")
    CombatManager.check_end()  # Uh oh, calling the manager from within...
    queue_free()
```

When the node starts knowing about 5 different systems, extract to manager. Node becomes clean again, manager handles coordination via signal.

### The Pointless Forwarding Anti-Pattern

```gdscript
# BAD — Manager adds no value
class_name ChestManager

func open_chest(chest: Chest) -> void:
    for item in chest.contents:
        GameSystems.inventory.add(item)

# The chest could just call inventory.add() directly!
```

If your manager only forwards to one system with no additional logic, **delete the manager**.

### Create a SYSTEM when...

Multiple parts of the game need the same operation:

```
HealthSystem needed by:
  - CombatManager (attack damage)
  - EnvironmentManager (fall damage, lava)
  - TrapManager (trap damage)
  - Node directly (self-damage abilities)
  
→ Extract to shared System
```

### DON'T create a System when...

Only one place uses it:

```gdscript
# day_cycle_manager.gd — no DayCycleSystem needed
# Only this manager touches day/night logic
func _advance_phase() -> void:
    current_phase = (current_phase + 1) % phases.size()
    phase_timer = 0.0
    phase_changed.emit(current_phase)
```

### Create a MANAGER when...

An event triggers effects across multiple unrelated systems:

```gdscript
# Enemy death touches: turns, loot, quests, achievements, combat state
# That's coordination — manager handles it
func _on_entity_died(enemy_id: int) -> void:
    GameSystems.turn.remove_participant(enemy_id)
    _roll_and_spawn_loot(enemy_id)
    QuestManager.notify_kill(enemy_id)
    AchievementManager.check(&"kills")
    _check_combat_end()
```

### DON'T create a Manager when...

The action only affects one system:

```gdscript
# BAD — SafeZoneManager just sets one flag
class_name SafeZoneManager
func enter() -> void:
    PlayerManager.set_safe(true)

# GOOD — Node calls directly, no middleman
func _on_area_entered(body: Node3D) -> void:
    PlayerManager.set_safe(true)
```

---

## The Design Process

> [!warning] Don't Architect Speculatively Don't build a "Dialogue System" until you actually have dialogue to implement. Start with features, not architecture.

### The Core Insight: You Skip Layers You Don't Need

The three-layer model is not a mandate—it's a toolkit. Many features skip straight to Node. That's fine. The architecture exists to _serve_ complexity, not to _create_ it.

**You don't always go Data → System → Manager → Node.**

You go: **Feature → [Do I need this layer?] → [Do I need this layer?] → Node**

### The Unified Process

```
┌─────────────────────────────────────────────────────────────────┐
│                    FEATURE WISHLIST                             │
│            "What does the player experience?"                   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│     Is there content/state that should be externalized?         │
│     (items, enemies, quests, skills, dialogue trees)            │
├─────────────────────────────────────────────────────────────────┤
│  YES → Design Definition/Instance (see Data-Driven Design doc)  │
│  NO  → Skip this layer                                          │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│     Are there operations multiple parts of the game need?       │
│     (damage, inventory add/remove, turn management)             │
├─────────────────────────────────────────────────────────────────┤
│  YES → Design System                                            │
│  NO  → Skip this layer                                          │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│     Does this event trigger effects across multiple systems?    │
│     (death → loot + quest + achievement + combat check)         │
├─────────────────────────────────────────────────────────────────┤
│  YES → Design Manager                                           │
│  NO  → Skip this layer                                          │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    IMPLEMENT NODES                              │
│     (Always needed — this is where the game actually runs)      │
└─────────────────────────────────────────────────────────────────┘
```

### Two Paths: Data-Driven vs. Behavior-Only

**Path 1: Data-Driven Feature**

Features where the "stuff" matters—inventory, enemies, items, quests, skills.

```
Feature: "Player can collect, store, and use items"
    ↓
Data: ItemDefinition (static), ItemInstance (runtime)
    ↓
System: InventorySystem (shared by loot, shops, crafting)
    ↓
Manager: Maybe PlayerManager for "use item" coordination
    ↓
Nodes: WorldItem (pickup), InventorySlot (UI)
```

**Path 2: Behavior-Only Feature**

Features with no "content" to externalize—movement, camera, simple interactions.

```
Feature: "Player moves with WASD, jumps with space"
    ↓
Data needed? No.
    ↓
Shared operations? No.
    ↓
Coordination needed? No.
    ↓
Result: Just a PlayerController node.
```

### Practical Starting Questions

When you pick up a feature from your wishlist:

|Question|If YES|If NO|
|---|---|---|
|Is there stuff a designer should edit without code?|Design data (Definition/Instance)|Skip data layer|
|Will multiple features need this same operation?|Create System|Keep logic in manager or node|
|Does this action ripple across unrelated systems?|Create Manager|Node calls system directly|

### Examples Across the Spectrum

|Feature|Data?|System?|Manager?|What You Build|
|---|---|---|---|---|
|Items/Inventory|✓|✓|Maybe|Full stack|
|Enemies with stats|✓|✓|✓|Full stack|
|Quest tracking|✓|✓|Maybe|Full stack|
|Player movement|✗|✗|✗|Just node|
|Door opens|✗|✗|✗|Just node|
|Safe zone|✗|✗|✗|Node → manager directly|
|Day/night cycle|Maybe|✗|✓|Manager + node|
|Camera shake|✗|Maybe|✗|Node or autoload|

### The Emergence Pattern

You often don't need architecture until complexity grows:

```gdscript
# Version 1: Simple — no extra layers
func die() -> void:
    queue_free()

# Version 2: A bit more — still just node
func die() -> void:
    _spawn_loot()
    queue_free()

# Version 3: Now touching 5 systems — time for a manager
func die() -> void:
    GameSystems.turn.remove(entity_id)
    _spawn_loot()
    QuestManager.notify(&"enemy_killed", definition.id)
    AchievementManager.increment(&"kills")
    CombatManager.check_end()
    queue_free()
# ^ This node knows too much. Extract coordination to manager.
```

**Start simple. Add layers when you feel the pain.**

---

## Project Structure

```
res://
├── systems/                    # Layer 0
│   ├── game_systems.gd        # Autoload container
│   ├── health_system.gd
│   ├── turn_system.gd
│   └── inventory_system.gd
│
├── managers/                   # Layer 1
│   ├── combat_manager.gd      # Autoload
│   ├── dialogue_manager.gd
│   └── player_manager.gd
│
├── helpers/                    # Technical isolation
│   └── targeting_helper.gd
│
├── data/                       # See Data-Driven Design doc
│   └── ...
│
└── scenes/                     # Layer 2
    ├── entities/
    ├── ui/
    └── world/
```

---

## Gotchas

### 1. Giant Match Statements

**Smell:** Manager has `match target.type:` with many cases.

**Fix:** Nodes handle themselves. `target.interact()` — node decides.

### 2. System Calling System

**Smell:** `HealthSystem` calls `TurnSystem.remove()` directly.

**Fix:** System emits signal, manager listens and coordinates.

### 3. Manager Doing Math

**Smell:** Damage formulas, raycast calculations in manager's public methods.

**Fix:** Extract to system (if shared) or helper (if messy single-use).

### 4. Over-Extraction

**Smell:** Created `PlayerSafetySystem` but only `PlayerManager` uses it.

**Fix:** Just a `var is_safe: bool` in the manager.

### 5. Presentation in Layer 0

**Smell:** System plays sounds or triggers animations.

**Fix:** System emits signal. Manager or node handles presentation.

### 6. Node Mutating Other Entities

**Smell:** `enemy.gd` does `player.health -= damage`.

**Fix:** `GameSystems.health.damage(player_id, amount)`.

### 7. Testing Blindness

**Test:** Can you write this without running the game?

```gdscript
func test_damage():
    var health := HealthSystem.new()
    health.register(1, 100)
    health.damage(1, 30)
    assert_eq(health.get_health(1), 70)
```

If you can't, your systems aren't properly isolated.

---

## Checklist

**Systems (Layer 0):**

- [ ] Extend `RefCounted`, not `Node`
- [ ] Zero scene tree dependencies
- [ ] Zero presentation (audio, visuals, UI)
- [ ] Communicate via signals only
- [ ] Used by multiple managers (that's why it's extracted)
- [ ] Can be unit tested without running game

**Managers (Layer 1):**

- [ ] Public methods read like English
- [ ] Private methods contain single-use logic
- [ ] Coordinate systems (don't duplicate their logic)
- [ ] React to system signals
- [ ] Tell nodes what to do (via direct call or signal)

**Nodes (Layer 2):**

- [ ] Handle their own type-specific behavior
- [ ] Own their visuals
- [ ] Call managers/systems for shared operations
- [ ] React to signals for updates
- [ ] Never mutate other entities' data directly

**Communication:**

- [ ] System → System: NEVER
- [ ] System → anything: Signals only
- [ ] Manager → System: Direct calls
- [ ] Node → System/Manager: Direct calls

---

## The Payoff

When you internalize layered architecture:

1. **Game logic is testable** — Run systems without scene tree
2. **Flow is readable** — Manager public API tells the story
3. **Changes are localized** — Swap visuals without touching logic
4. **Bugs are traceable** — Clear signal flow, visible coordination
5. **Adding types is easy** — New node script, manager unchanged
6. **Godot is your ally** — Nodes handle themselves, polymorphism over switch statements

> [!quote] The Ultimate Test Can you explain your game's flow by reading just the manager's public methods?
> 
> If yes, you've achieved clean architecture. If no, details are leaking into orchestration.