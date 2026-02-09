# Layered Architecture

---

## You're Already at Level 1

If you're building with components — a Health script on your enemy, a Door script on your door, each owning its own state — you're already doing the foundation. That's **Component-as-Truth**, and it's where most of your game lives. This doc covers two additional tools you add on top when specific problems arise.

### Add a System when:

- **You need to query or batch-operate across entities** — "find all enemies below 50% HP", "heal all allies" — and there's no central place to ask
- **Data has a Definition/Instance lifecycle** — an InventorySystem managing item instances created from data templates (see [[Data-Driven Design]])
- **Save/load needs one source** — persisting all entity state from a single place instead of scattered components
- **Multiplayer needs authority** — server needs one authoritative source of truth for entity state

### Add a Manager when:

- **One event triggers multiple unrelated things and order matters** — enemy dies → remove from turn order, THEN roll loot, THEN notify quests, THEN check combat end
- **Logic needs engine features but isn't type-specific** — a day/night cycle that needs timers, update loops, and scene access

If neither of these apply, Component-as-Truth is the right answer. **Most game objects are just components owning themselves.**

> [!info] "Layered" is a common name, but it's not strict layering (each layer only talks to the one below). Think of it as **three roles with rules**, not a rigid stack.

---

## The Three Roles

```
┌───────────────────────────────────────────────────────────────┐
│                        SYSTEMS                                 │
│                    "What IS true?"                             │
├───────────────────────────────────────────────────────────────┤
│  Pure data and logic. Zero visuals/UI/sound. Isolated.         │
│  Plain classes. Announces changes via events/signals.          │
│  Testable without the engine running.                          │
│                                                                │
│  HealthSystem knows health values and damage formulas.         │
│  Knows NOTHING about death animations or loot drops.           │
└───────────────────────────────────────────────────────────────┘
                            │
                            │ Managers CALL systems
                            │ Systems EMIT events/signals
                            ▼
┌───────────────────────────────────────────────────────────────┐
│                        MANAGERS                                │
│                "What happens when X occurs?"                   │
├───────────────────────────────────────────────────────────────┤
│  Orchestration. Coordinates ripple effects across the game.    │
│  Connects systems to presentation. May need engine features    │
│  (timers, scene access, spawning, update loops).               │
│                                                                │
│  CombatManager hears "died" event →                            │
│  removes from turn order, rolls loot, notifies quests.         │
└───────────────────────────────────────────────────────────────┘
                            │
                            │ Managers TELL components
                            │ Components CALL managers/systems
                            ▼
┌───────────────────────────────────────────────────────────────┐
│                       COMPONENTS                               │
│                "I know what X means for MY type"               │
├───────────────────────────────────────────────────────────────┤
│  Type-specific behavior. Owns visuals. Handles itself.         │
│  Built with composition — each component does one job.         │
│                                                                │
│  Door knows what "interact" means for a door.                  │
│  Manager just says "interact()", door figures out the rest.    │
└───────────────────────────────────────────────────────────────┘
```

| Role | Analogy | Responsibility |
|---|---|---|
| **System** | The Script | What IS true |
| **Manager** | The Director | WHEN things happen, WHO does WHAT |
| **Component** | The Actor | HOW to perform its part |

---

## Source of Truth

**"Truth"** = where authoritative state lives. The single place that knows the real value. Everything else reads from the truth owner or requests changes through it.

| Owner | When | Example |
|---|---|---|
| **System** | Shared pool or externalized data | HealthSystem owns all entity HP |
| **Component** | Self-contained, local state | Door owns its open/closed state |
| **Manager** | Rare — game-wide state only it tracks | DayCycleManager owns current phase |

### Reading and Mutating

**Reading** — always fine, go directly to the source:
```
// System owns it:
GameSystems.health.get(entityId)

// Component owns itself:
player.position
```

**Mutating** — request the action, don't reach into internals:
```
// System owns it:
GameSystems.health.damage(entityId, 10)

// Component owns itself:
enemy.takeDamage(10)

// BAD — bypasses events, validation, everything:
enemy.health -= 10
```

---

## The Default: Component-as-Truth

Most game objects don't need Systems. A door, a barrel, a pickup, a decoration — they own their own state. **This is the default pattern. Start here.**

```
class Health:
    max: int = 100
    current: int

    event onChanged(newValue)
    event onDepleted()

    awake():
        current = max

    takeDamage(amount):
        current = max(0, current - amount)
        onChanged.emit(current)
        if current == 0:
            onDepleted.emit()
```

Other components on the same object react to these local events:

```
class DeathEffect:
    health: Health    // sibling component reference

    onEnable():
        health.onDepleted.subscribe(handleDeath)

    handleDeath():
        playParticles()
        destroySelf(afterDelay: 2)
```

Assembling entities through composition:

```
"Enemy"                    "Barrel"
├── Health (100 HP)        ├── Health (30 HP)
├── DeathEffect            ├── DeathEffect
├── LootDropper            └── (done)
└── EnemyAI

"Boss"
├── Health (5000 HP)
├── DeathEffect
├── LootDropper
├── BossAI
└── BossHealthBar          // reads Health.current for UI
```

Health, DeathEffect, LootDropper are **reusable** — they work on enemies, barrels, NPCs, bosses, anything. EnemyAI and BossAI are **entity-specific**. Aim for roughly 70% reusable, 30% entity-specific glue.

Events are local — `Health.onDepleted` fires on that specific object. Only listeners on that object hear it. No filtering, no wasted invocations.

---

## When and Why to Centralize (Add a System)

A System exists for one of two reasons:

**1. Shared pool.** One place holds state for multiple entities — `HealthSystem` stores `{entity_1: 100, entity_2: 50, entity_3: 80}`. You need this when you need to:
- Query across entities ("find all below 50% HP")
- Batch operate ("heal all allies")
- Save/load from one place
- Enforce server authority in multiplayer

**2. Externalized data (Data/State pattern).** A System manages runtime instances created from data templates — `InventorySystem` creates ItemInstances from ItemDefinitions. See [[Data-Driven Design]].

### What "Shared" Does NOT Mean

**"Multiple places call it" is NOT enough for a System.** The question is who owns the data.

```
MULTIPLE CALLERS, LOCAL STATE → NOT a System:
  Each entity has its own Health component
  Combat, traps, environment all call health.takeDamage()
  Multiple callers ≠ shared state. Each component owns itself.

SHARED POOL → System:
  HealthSystem stores {entity_1: 100, entity_2: 50}
  One place owns the data for ALL entities.
```

If you need shared **logic** but not shared **state**, use a static helper — not a System:

```
// Shared LOGIC, no shared STATE → helper
class DamageCalculator:
    static calculate(baseDamage, armor, multiplier):
        return max(1, round((baseDamage - armor) * multiplier))
```

### Component-as-View: When a System Owns Truth

When you centralize to a System, components become bridges between the System and the visual object. They register/unregister, display system state, and request mutations through the system API:

```
class HealthRegistration:
    entityId: int
    maxHealth: int = 100

    onEnable():
        entityId = getInstanceId()
        GameSystems.health.register(entityId, maxHealth)

    onDisable():
        GameSystems.health.unregister(entityId)

    currentHealth -> int:
        return GameSystems.health.get(entityId)
```

> [!warning] Shotgun Events With a centralized system, death handlers subscribe to a global `onDied` event and filter by ID. With 200 entities, every death triggers 200 invocations — 199 immediately return. Component-as-Truth (`Health.onDepleted`) has no filtering — only local listeners hear it. This is a real cost of centralization.

---

## When and Why to Add Managers

Managers coordinate when an event has **ripple effects across multiple unrelated things** and **order matters**.

```
class CombatManager:
    onEnable():
        GameSystems.health.onDied.subscribe(handleDeath)

    handleDeath(entityId):
        // Orchestrate IN ORDER
        removeFromTurnOrder(entityId)
        spawnLoot(entityId)
        notifyQuests(entityId)
        checkCombatEnd()
```

**Manager is better than "everyone listens to signals" when:**
- Order matters (remove from turn BEFORE checking combat end)
- Conditional logic ("if boss, trigger cutscene")
- You need one place to trace/debug the flow

**Manager is overkill when:**
- Listeners are independent
- Order doesn't matter
- Fire and forget

If a "Manager" just forwards a single call — delete it and call directly.

**What Managers can do that Systems can't:** coroutines/timers, scene access, update loops, spawn objects. If logic needs engine features but isn't type-specific, it's a Manager.

---

## Communication Rules

```
┌─────────────────────┬─────────────────────────────────────┐
│  FROM → TO          │ METHOD                               │
├─────────────────────┼─────────────────────────────────────┤
│  Component → System │ Direct call                           │
│  Component → Manager│ Direct call or event                  │
│  Manager → System   │ Direct call                           │
│  Manager → Component│ Direct call (if owned) or event       │
│  System → anything  │ EVENT/SIGNAL ONLY                     │
│  System → System    │ NEVER                                 │
└─────────────────────┴─────────────────────────────────────┘
```

**Why Systems only emit events:** They're pure truth machines — hold data, apply rules, announce changes. That's it.

**Why Systems never call Systems:** The moment SystemA listens to SystemB, it's no longer just "truth about A." That coordination is a Manager's job. Otherwise you get hidden dependencies, uncontrolled execution order, and scattered logic.

---

## The Decision

```
Q1: Does this feature have externalized data (Data/State pattern)?
    YES → System owns the runtime instances

Q2: Do you need a shared pool? (querying, batch ops, persistence, multiplayer)
    YES → System owns the pool

Q3: Does this event ripple across multiple unrelated things?
    YES → Manager coordinates

Q4: Does logic need engine capabilities (timers, update loop, spawning)?
    YES → Manager or Component
    NO  → Static helper (if shared logic) or Component-as-Truth

Q5: None of the above?
    → Component-as-Truth (owns itself)
```

| Feature | Externalized Data? | Shared Pool? | Ripple? | Result |
|---|---|---|---|---|
| Inventory | Yes | Yes | Maybe | System + Component-as-View |
| Quests | Yes | Yes | Maybe | System + Component-as-View |
| Health (need queries/batch/save) | No | Yes | — | System + Component-as-View |
| Health (simple per-entity) | No | No | — | Component-as-Truth |
| Door interaction | No | No | No | Component-as-Truth |
| Barrel | No | No | No | Component-as-Truth (Health + DeathEffect) |
| Enemy death effects | — | — | Yes | Manager coordinates |
| Day/night cycle | No | No | Yes | Manager (needs timers, update) |
| Damage formula | — | Shared logic only | — | Static helper |

---

## Components Handle Themselves

Managers shouldn't need to know what type of object they're talking to. Objects define their own behavior through shared interfaces:

```
interface Interactable:
    interact()
    getPrompt() -> string

// BAD — Manager knows all types
handleInteraction(target):
    if target is Door: openDoor(target)
    else if target is Chest: openChest(target)

// GOOD — Component decides
handleInteraction(target):
    if target implements Interactable:
        target.interact()
```

New interactable type = new script implementing the interface. No manager changes.

---

## Keep It Simple

> [!warning] Architecture is a tool, not a religion.

| Smell | Reality |
|---|---|
| "System because multiple things call it" | Is state in a shared pool? If no, Component-as-Truth. |
| "Manager" that forwards one call | Delete it. Call directly. |
| "Event bus" to read player position | Just read it. |
| System for everything | Most things are Component-as-Truth. |
| Giant switch/match on type in Manager | Components should handle themselves. |

Most game objects are simple. A door, a trigger, a decoration — just owns itself. Architecture exists for genuinely complex cases: shared pools, externalized data, ripple effects, persistence.

**Start with Component-as-Truth. Add Systems and Managers when complexity demands it — not before.**

---

## Key Takeaways

1. **Most games don't need formal architecture** — objects owning themselves is the default, and it's fine
2. **Systems** = pure truth (plain classes, events only) — only for shared pools or externalized data
3. **Managers** = orchestration — only when ripple effects need ordering
4. **Components** = actors, built with composition — **this is where most of your game lives**
5. **Systems exist for two reasons** — shared pool or externalized data (not "multiple callers")
6. **Components handle themselves** — interfaces, not switch statements
7. **Add layers when you feel the pain, not before**

> [!quote] The Test (when using this pattern): Can you explain your game's flow by reading just the manager's public methods?

---

See [[Data-Driven Design]] for the Data/State/Representation pattern that Systems often manage.
