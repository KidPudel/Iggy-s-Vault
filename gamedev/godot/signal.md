Signal in Godot is basically a pub/sub patter like [[event bus]], where you have a signal, that you can emit, and subscribed/connected nodes via functions gets notified (connected function gets called).

Perfect for side effect and cascade calls.

Exposes, makes it global and common

In Godot since [[node in godot]] are components, that are by design is better to kept more isolated (mind your own business). It is a common practice to follow [[Call down, signal up]]. Which turns direction communication to the lazy and everyone is happy.
But why call down? because down, means to its children, meaning that the children of the node is the part of the node. It is the [[scene]] - the composition of the nodes.


Mental model: **Call systems directly for commands. Listen to signals for reactions.**

```python
# ✅ COMMAND - Direct call
inventory_system.add(item)
combat_system.attack(weapon, target)
crafting_system.craft(recipe)

# ✅ REACTION - Signal listener
inventory_system.item_added.connect(_on_item_added)
combat_system.damage_dealt.connect(_on_damage_dealt)
```

REMEMBER: Signal catches everyone who is connected. Check accordingly.