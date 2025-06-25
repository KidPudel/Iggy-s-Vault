ddd is a quality improvement approach where you introduce the aspect where code can be fried from making decisions. Data + state + systems - decision makers

you extract what can be stored as:
- [[declarative]]/static data. content = raw data describing what exists in the world
- Parametrized behavior, data value, that influence on runtime logic. Behavior = logic that is data-driven

Note: store data that is appropriate

The rest is better to stay in code



# Difference
- Code-driven: logic and flow are embedded in the code. State is often implicit
- Data-driven: **logic reacts to data**, meaning **data defines behavior**. Systems consume state/config and produce the output




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

