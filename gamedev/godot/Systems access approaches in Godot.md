
Three ways to access your game systems. Pick based on project size and team needs.

---

## 1. Multiple Autoloads

Every system is a separate [[autoload]].

```gdscript
# Project Settings → Autoload:
# - Inventory (inventory_system.gd)
# - Combat (combat_system.gd)
# - Quests (quest_system.gd)
```

```gdscript
# inventory_system.gd
extends Node

signal item_added(item: ItemInstance)

var items: Array[ItemInstance] = []

func add(item: ItemInstance) -> void:
    items.append(item)
    item_added.emit(item)
```

**Usage anywhere:**

```gdscript
Inventory.add(item)
Combat.attack(weapon, target)
Quests.complete("quest_01")
```

|Pros|Cons|
|---|---|
|Zero setup|Global mutable state everywhere|
|Access from anywhere|Hard to track who modifies what|
|Fast prototyping|Tight coupling to global names|
||Can't easily reset/swap systems|
||Testing requires mocking globals|

**Best for:** Game jams, prototypes, small solo projects.

---

## 2. Single Container Autoload (Middle Ground)

One autoload holds all system instances.

```gdscript
# game.gd (single Autoload named "Game")
extends Node

var inventory: InventorySystem
var combat: CombatSystem
var quests: QuestSystem

func _ready() -> void:
    inventory = InventorySystem.new()
    combat = CombatSystem.new()
    quests = QuestSystem.new()

func reset_all() -> void:
    inventory = InventorySystem.new()
    combat = CombatSystem.new()
    quests = QuestSystem.new()
```

```gdscript
# inventory_system.gd (not a Node, just logic)
class_name InventorySystem extends RefCounted

signal item_added(item: ItemInstance)

var items: Array[ItemInstance] = []

func add(item: ItemInstance) -> void:
    items.append(item)
    item_added.emit(item)
```

**Usage anywhere:**

```gdscript
Game.inventory.add(item)
Game.combat.attack(weapon, target)
Game.quests.complete("quest_01")
```

|Pros|Cons|
|---|---|
|One global entry point|Still global access|
|Systems can be reset/swapped|Slightly more typing|
|Clearer what exists|Still some coupling|
|Systems aren't Nodes (cleaner)||
|Easy to add new systems||

**Best for:** Small-to-medium projects, solo or small teams.

---

## 3. Dependency Injection

No autoloads. Scene root creates and passes systems.

```gdscript
# main.gd (scene root)
extends Node

func _ready() -> void:
    # Create systems
    var inventory := InventorySystem.new()
    var combat := CombatSystem.new()
    var quests := QuestSystem.new()
    
    # Inject into nodes that need them
    $Player.setup(inventory, combat)
    $UI/InventoryUI.setup(inventory)
    $UI/QuestLog.setup(quests)
    $World.setup(inventory)  # for item pickups
```

```gdscript
# player.gd
class_name Player extends CharacterBody3D

var inventory: InventorySystem
var combat: CombatSystem

func setup(inv: InventorySystem, cbt: CombatSystem) -> void:
    inventory = inv
    combat = cbt
```

```gdscript
# inventory_ui.gd
class_name InventoryUI extends Control

var inventory: InventorySystem

func setup(inv: InventorySystem) -> void:
    inventory = inv
    inventory.item_added.connect(_on_item_added)
```

**Usage (only where injected):**

```gdscript
# In player.gd
inventory.add(item)

# In inventory_ui.gd
for item in inventory.items:
    _create_slot(item)
```

|Pros|Cons|
|---|---|
|No globals|More wiring code|
|Explicit dependencies|Must pass systems through tree|
|Easy to test/mock|Can't access from "anywhere"|
|Easy to reset (just recreate)|Requires planning|
|Scenes are self-contained||

**Best for:** Medium-to-large projects, teams, when testability matters.

---

**Rule of thumb:**

- **Read-only / stateless** → Autoload is fine
	- **Data or functions that don't change during gameplay.**
- **Mutable game state** → Inject or container
	- **Data that changes during gameplay and affects game logic.**

---

## Warning Signs

**Too many autoloads:**

```gdscript
# If you see this pattern everywhere, reconsider
Inventory.add(item)
PlayerStats.add_xp(100)
QuestManager.update("quest_01")
AudioManager.play("pickup")
EventBus.emit("item_picked_up", item)
Analytics.track("pickup", item.id)
```

Problems:

- Any script can modify anything
- Hard to trace bugs
- Difficult to reset game state
- Testing becomes painful

**Solution:** Group related systems, inject mutable state.

---

## Summary

| Pattern              | Access                 | Coupling | Testing  | Best For              |
| -------------------- | ---------------------- | -------- | -------- | --------------------- |
| Multiple Autoloads   | `System.method()`      | High     | Hard     | Jams, prototypes      |
| Single Container     | `Game.system.method()` | Medium   | Moderate | Small-medium projects |
| Dependency Injection | `system.method()`      | Low      | Easy     | Medium-large, teams   |

All three are valid. Match the pattern to your project's needs. You can always refactor from autoloads to injection later — the system logic stays the same, only how you access it changes.