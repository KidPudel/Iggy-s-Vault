1. `queue_free()` for Nodes:
- This function is called on a Node instance (e.g., `my_node.queue_free()`).
- It marks the node for deletion at the end of the current frame, ensuring proper cleanup and preventing errors that might occur if a node is immediately freed while other parts of the game are still trying to access it.
- When `queue_free()` is called on a parent node, all of its child nodes are also freed automatically.

2. `free()` for Objects (and specific Node cases):
- The `free()` function immediately deletes an `Object` instance.
- It is necessary for `Object` instances that do not manage their own memory, like custom classes inheriting directly from `Object` (not `Node`), to prevent memory leaks.
- While `free()` can be used on nodes, `queue_free()` is generally preferred for nodes within the scene tree to avoid potential errors related to immediate deletion.
- Using `free()` on a node that is not in the scene tree or has no children is generally safe.


- **Validity Checks:**  When referencing objects that might be deleted, it's good practice to use `is_instance_valid(reference_to_object)` to check if the object still exists before attempting to access its properties or methods. or `node.is_marked_for_deletion()`

- **Scene Deletion:**  To unload an entire scene, you can get a reference to the current scene (e.g., `get_tree().get_current_scene()`) and remove it as a child from the root node, which will effectively free all its nodes.