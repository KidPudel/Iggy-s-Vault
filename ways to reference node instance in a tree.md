## Static Access (known at design time)

1. Call it directly with a string path `get_node("path")`. This is hardcoded, but works for static, predictable trees.

2. Use shorthand sugar syntax `$Node` or `%NodeName` with help of [[class in godot]]. `$Node` is the same as `get_node("Node")`, relative to the current node. `%NodeName` refers to uniquely named nodes in the scene [[Unique node]].

3. Access the parent node with `get_parent()` or `get_node("..")`. Useful when a node needs to reach its container or controller.

4. Use absolute path from the root `get_tree().get_root().get_node("full/path")`. Allows global access to nodes but is fragile if the scene structure changes. [[SceneTree]]

---

## Editor-Assigned Access (Inspector-driven)

5. Assign a node directly via Inspector using `@export var target: Node`. This gives a typed reference to an existing node without hardcoding. [[export in godot]]

6. Store a path to the node using `@export var path: NodePath` and resolve it with `get_node(path)`. More decoupled than direct references and allows Inspector configuration.

---

## Runtime Access (dynamic or indirect)

7. Use [[groups]] with `get_tree().get_nodes_in_group("group_name")`. Lets you reference one or many nodes sharing a role or behavior.

8. Perform a recursive search with `find_child("Name")` or `find_node("Name", recursive = true)`. Useful for uncertain hierarchy but slower and less safe.

9. Use `owner` to reference the instancer of the current node. Helpful when a scene is instanced inside another and needs to call back to its context.

10. Pass the reference directly via method arguments. Keeps things decoupled and clear when the reference is already available in the calling scope.

11. Receive the node via signal arguments. Emit `self` or another node in a signal, and let the connected listener receive and interact with it. [[signal]]

---

## Global Access

12. Use a global singleton via Autoload (defined in Project Settings → Autoload). Provides central access to managers or persistent references like `Global.player_node`.
