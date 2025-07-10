Node in Godot is a hybrid of data and behavior unit.
It is a [[reference counter]], engine-managed handle (wrapper) to an underlying scene tree object.
It contains:
- [[metadata]] (name, links)
- Built-in lifecycle hooks (`_ready`, `_process`)
- Optionally attached to it scripts, signals, etc

It is a [[heap]] allocated object, that has a lifecycle of a scene in a game.
Medium [[Memory fragmentation]]
[[cache locality]] is low because of tree traversal

A collection of nodes is called [[scene]]

# Node's readiness
Node triggering when both node and its children entered a tree and all children's `_ready` are already finished
It is called deferred

Tree entered ->
Ready <-
This is why [[Call down, signal up]]
If we still want to access the parent components of node in a scene, in `_ready`, we can call `call_deferred("_func_to_run")`, which will run at the idle time.
Idle time happens at the end of the process and physics frames.


# Dependency. Component nature.
Since nodes are components, it is best to keep them self-contained as possible, this removes the tight-coupling.
Nodes that has children owns its children, meaning interaction with it is normal
[[signal]] allowing children to keep their reaction, while not introducing tight coupling.
Outer nodes are the parent of the child nodes.


---

# Ways to reference node instance in a tree
[[ways to reference node instance in a tree godot]]
