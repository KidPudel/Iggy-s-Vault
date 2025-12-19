**Data-driven design** = data is the source of truth, code operates on it, visuals display it.
It is about data separation and isolation, which helps create maintainable and sane space.
Programming can be described as "semantic compression": taking all the work that computer must do and expressing it in the most compressed, non-duplicated way that a human can still understand and maintain.

An RPG's items, enemies, quests, dialogue — that _is_ the game/problem you are solving. The code is just an interpreter. When you externalize data into resources/data, you're acknowledging this: the data is the creative artifact (fuel), code is infrastructure (engine).

Same principle as: HTML is the document (fuel), browser is the renderer (engine). You don't embed the document inside the browser's source code.

ddd is a quality improvement approach where you introduce the aspect where code can be fired from making decisions. Data + state + systems - decision makers

you extract what can be stored as:
- [[declarative]]/static data. content = raw data describing what exists in the world
- Parametrized behavior, data value, that influence on runtime logic. Behavior = logic that is data-driven

# Difference
- Code-driven: logic and flow are embedded in the code. State is often implicit
- Data-driven: **logic reacts to data**, meaning **data defines behavior** - it is the fuel. Systems consume state/config and produce the output - it is the engine

## What you gain:
- **Change content without changing code** — balance a weapon, add an item, tweak enemy stats. No recompile, no risk of breaking logic.
- **Data is readable** — open a folder, see every item in your game as files. The game's content is browsable, diffable, searchable.
- **Multiple access points** — editor, code, tools, external scripts can all read/write the same data. Centralized truth point, the rest are "users" of the data/code
- **Clear boundary** — "what exists" is separate from "how it behaves."

> Systems: logic processors, they do the actual work, but **only by reading/modifying data**, not by holding logic state themselves, and not directly decision makers (the behaviors and states are).


```python
class BehaviorSystem:
    def update(self, entity, state, behavior_config):
        for rule in behavior_config:
            if self.evaluate(rule["if"], state[entity]):
                self.execute(rule["do"], entity, state)
                break

    def evaluate(self, condition, entity_state):
        if condition == "hp < 50":
            return entity_state["hp"] < 50
        if condition == "enemy_in_range":
            return self.check_proximity(entity_state)
        if condition == "true":
            return True

    def execute(self, action, entity, state):
        if action == "flee":
            state[entity]["state"] = "running_away"
        elif action == "attack":
            state[entity]["state"] = "attacking"
        elif action == "patrol":
            state[entity]["state"] = "patrolling"
```


[[data-driven design in godot with architecture in mind]]