# Layered Architecture in Godot

> [!quote] The Core Insight Separate what IS true (simulation) from what SHOWS the truth (presentation).

---

## The Problem

Without architecture, code becomes a web where everything depends on everything. Change one thing, break ten others. Can't test in isolation. Can't reason about flow.

**The fix:** Layers with clear responsibilities and communication rules.

---

## The Three Layers

```
┌─────────────────────────────────────────────────────────────────┐
│                        SYSTEMS                                  │
│                    "What IS true?"                              │
├─────────────────────────────────────────────────────────────────┤
│  Pure data and logic. Zero visuals/UI/sound.                    │
│  Extends RefCounted. Emits signals. Testable without scene.     │
│                                                                 │
│  Example: HealthSystem knows health values and damage formulas. │
│  Knows NOTHING about death animations or loot drops.            │
└─────────────────────────────────────────────────────────────────┘
                              │
                              │ Managers CALL systems
                              │ Systems EMIT signals
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                        MANAGERS                                 │
│                "What happens when X occurs?"                    │
├─────────────────────────────────────────────────────────────────┤
│  Game-specific choreography. Coordinates ripple effects.        │
│  Extends Node. Connects systems to presentation.                │
│                                                                 │
│  Example: CombatManager hears "died" signal →                   │
│  removes from turn order, rolls loot, notifies quests.          │
└─────────────────────────────────────────────────────────────────┘
                              │
                              │ Managers TELL nodes
                              │ Nodes CALL managers/systems
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                         NODES                                   │
│                "I know what X means for MY type"                │
├─────────────────────────────────────────────────────────────────┤
│  Type-specific behavior. Owns visuals. Handles itself.          │
│  Manager doesn't need to know internals.                        │
│                                                                 │
│  Example: Door knows what "interact" means for a door.          │
│  Manager just says "interact", door figures out the rest.       │
└─────────────────────────────────────────────────────────────────┘
```

|Layer|Analogy|Responsibility|
|---|---|---|
|**System**|The Script|What IS true|
|**Manager**|The Director|WHEN things happen, WHO does WHAT|
|**Node**|The Actor|HOW to perform its part|

---

## Source of Truth

**"Truth"** = where authoritative state lives. The single place that knows the real value.

### Who Can Own Truth

|Owner|When|Example|
|---|---|---|
|**System**|Externalized data (Definition/Instance) or shared pool|`InventorySystem` owns all items|
|**Node**|Self-contained, local state|`Door` owns its open/closed state|
|**Manager**|Rare—game-wide state only it tracks|`DayCycleManager` owns current phase|

### Two Kinds of Nodes

|Kind|Truth lives...|Node's job|
|---|---|---|
|**Node-as-Truth**|In itself|Own state, BE the feature|
|**Node-as-View**|In a System|Display state, request mutations|

Most game objects (doors, trees, triggers, decorations) are **Node-as-Truth**. They don't need Systems—they ARE the feature.

**Node-as-View** exists when a System owns the data:

```
HealthSystem owns entity health
    ↓
HealthBar displays it (subscribes to signals, reads System)
```

### The Core Principles

```
READING:      From wherever truth lives
MUTATING:     Request action to the owner
COORDINATING: Through Managers (for ripple effects)
```

**Reading** — always fine. Read from the source:

- System owns it → `GameSystems.health.get(id)`
- Node owns itself → `player.global_position`

**Mutating** — request action, don't reach into internals:

```gdscript
# If System owns it:
GameSystems.health.damage(id, 10)

# If Node owns itself:
enemy.take_damage(10)

# BAD — reaching into internals:
enemy.health -= 10
```

---

## When to Create What

> [!warning] Don't Architect Speculatively Start simple. Add layers when complexity demands it.

### The Decision

```
Q1: Does this feature have externalized data (Definition/Instance)?
    YES → System owns that data
    NO  → Continue...

Q2: Is state centralized in a shared pool?
    YES → System owns the pool
    NO  → Node owns itself

Q3: Does this event ripple across multiple unrelated systems?
    YES → Manager coordinates
    NO  → Continue...

Q4: Does logic need Node capabilities (timers, tree, process)?
    YES → Manager (or Node-as-Truth if view-specific)
    NO  → System or helper
```

### Why Systems

|Reason|Example|
|---|---|
|Externalized data|`InventorySystem` owns ItemInstances|
|Shared pool|`HealthSystem` stores `{entity_id → hp}` for all entities|

Systems are `RefCounted`—pure logic, no scene tree, testable in isolation, they are purely truth machines.

### Why Managers

|Reason|Example|
|---|---|
|**Coordination**|Enemy death → turn order + loot + quests + combat end|
|**Logic home**|Needs Node capabilities but shouldn't live in view|

**Manager vs Signals Everywhere:**

You could skip managers—everyone listens to signals directly. That works when:

- Order doesn't matter
- Listeners are independent
- Fire and forget

Manager is better when:

- Order matters (remove from turn BEFORE checking combat end)
- Conditional logic ("if boss, trigger cutscene")
- One place to trace/debug the flow

**Manager capabilities Systems lack:**

|Capability|System|Manager|
|---|---|---|
|Timers / await|✗|✓|
|Scene tree access|✗|✓|
|`_process` / `_physics_process`|✗|✓|
|Own child nodes|✗|✓|
|Spawn scenes|Awkward|✓|

### What "Shared" Actually Means

"Multiple places call it" is **not enough** for a System.

|Term|Meaning|System?|
|---|---|---|
|**Shared pool**|One place holds state for multiple entities|Yes|
|**Multiple callers**|Different places call the same operation|Not necessarily|

```
SHARED POOL → System:
HealthSystem stores {entity_1: 100, entity_2: 50}
Combat, traps, environment all call HealthSystem.damage()

LOCAL STATE → Node-as-Truth:
Each Enemy owns `var health: int`
Combat, traps, environment all call enemy.take_damage()
Same callers, but each node owns itself.
```

If you need shared **logic** but not shared **state**, use a helper—not a System.

### Quick Reference

|Has Data?|Shared Pool?|Ripple?|Needs Node?|Result|
|---|---|---|---|---|
|Yes|—|—|—|System|
|No|Yes|—|—|System|
|No|No|Yes|—|Manager|
|No|No|No|Yes|Manager or Node|
|No|No|No|No|Node-as-Truth or Helper|

### Examples

|Feature|Shared Pool?|Ripple?|Result|
|---|---|---|---|
|Inventory|Yes (items)|Maybe|System + Node-as-View|
|All-entity health|Yes|—|HealthSystem|
|Per-enemy health|No (each owns)|—|Node-as-Truth|
|Door interaction|No|No|Node-as-Truth|
|Stick scatter|No|No|Node-as-Truth|
|Enemy death|—|Yes|Manager coordinates|
|Day/night cycle|No|Yes|Manager (also needs timers)|
|Turn timer|No|No|Manager (needs _process, timers)|

---

## Communication Rules

### Control Flow

```
┌─────────────────────┬───────────────────────────────────────────┐
│  FROM → TO          │ METHOD                                    │
├─────────────────────┼───────────────────────────────────────────┤
│  Node → System      │ Direct call                               │
│  Node → Manager     │ Direct call                               │
│  Manager → System   │ Direct call                               │
│  Manager → Node     │ Direct call (owned) OR signal             │
│  System → anything  │ SIGNAL ONLY                               │
│  System → System    │ NEVER                                     │
└─────────────────────┴───────────────────────────────────────────┘
```

**Why Systems only emit signals:** They're pure truth—announce changes, don't control flow.

---

**Why Systems never call Systems:** Hidden dependencies. Manager coordinates instead.

Systems shouldn't listen to other systems because *systems are supposed to be pure truth machines*. They:
- Hold data
- Apply rules
- Announce changes
- **That's it**

> [!danger] The moment `PlayerSystem` listens to `CampfireSystem`, it's no longer just *"truth about player."*. That's not a System's job. That's a **Manager's** job.

Otherwise we get:
1. dependency on others
2. uncontrolled flow and order of it
3. scattered logic

Everything becomes more complicated than it is.

---
### Manager → Node Communication

|Relationship|Mechanism|
|---|---|
|Manager owns the node|Direct: `$CombatUI.show()`|
|Global service|Autoload: `AudioManager.play()`|
|Specific node needed|Stored reference or group|
|Loose coupling|Signal|

### Nodes Handle Themselves

```gdscript
# BAD — Manager knows all types
func interact(target):
    match target.type:
        "door": ...
        "chest": ...

# GOOD — Node decides
func interact(target):
    target.interact()
```

Add new type = add new script. Manager unchanged.

---

## Keep It Simple

> [!warning] Architecture is a tool, not a religion.

**Signs you're over-architecting:**

|Smell|Reality|
|---|---|
|"System because multiple things call it"|Is state in a shared pool? If no, Node-as-Truth.|
|"Manager" but it just forwards one call|Call directly.|
|"Event bus" to read player position|Just read it.|
|"Signals" for parent-child|Direct reference is fine.|

**Most features are simple.** A door, a trigger, a decoration—just Node-as-Truth. The architecture exists for genuinely complex cases: shared pools, ripple effects, persistence.

**Example — Stick Scatter:**

Multimesh of sticks, moves collision box to track player.

```
Shared pool? No.
Persistence? No.
Ripple effects? No.
```

Result: Node-as-Truth. Reads `player.global_position` directly. Done.

---

## Quick Reference

### Project Structure

```
res://
├── systems/              # RefCounted, pure logic
│   ├── game_systems.gd   # Autoload container
│   ├── health_system.gd
│   └── inventory_system.gd
├── managers/             # Node, coordination
│   ├── combat_manager.gd
│   └── day_cycle_manager.gd
├── helpers/              # Shared logic (not shared state)
│   └── damage_calculator.gd
└── scenes/               # Nodes
    ├── entities/
    ├── ui/
    └── world/
```

### Gotchas

|Smell|Fix|
|---|---|
|Manager has giant `match type:`|Nodes handle themselves|
|System calls System|Signal → Manager coordinates|
|System plays sounds/animations|Signal → presentation handles|
|`enemy.health -= 10`|Request action: `enemy.take_damage(10)`|
|Can't test without running game|Systems not isolated|
|System for "multiple callers"|Is state pooled? If no, wrong reason.|
|Manager for independent listeners|If order doesn't matter, signals alone work|

### Checklist

**Systems:**

- [ ] `RefCounted`, zero scene dependencies
- [ ] Zero presentation
- [ ] Signals only (output)
- [ ] Exists because: externalized data OR shared pool

**Managers:**

- [ ] Exists because: ripple effects OR needs Node capabilities
- [ ] Public methods read like English
- [ ] Reacts to system signals (if coordinating)

**Nodes:**

- [ ] Node-as-Truth: owns state, IS the feature
- [ ] Node-as-View: displays, requests mutations
- [ ] Handles its own type-specific behavior

**Sanity:**

- [ ] Could the node just own itself?
- [ ] Is there actually a shared pool?
- [ ] Could signals alone handle this? (No order/conditionals needed?)

---

## The Payoff

1. **Testable** — Systems run without scene tree
2. **Readable** — Manager tells the story
3. **Traceable** — Clear signal flow
4. **Extensible** — New type = new script

> [!quote] The Ultimate Test Can you explain your game's flow by reading just the manager's public methods?

---

_For data structure (Definition/Instance pattern), see [[Data-Driven Design in Godot]]._