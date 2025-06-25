In godot [[viewport]] is both:
- Rendering target - surface where scenes are drawn
- A node (Viewport) - container that renders its own scene tree, isolated from main scene tree.

by using `some_ref_to_node.get_viewport()`, allows you to get the closest `Viewport` ancestor