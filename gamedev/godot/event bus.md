[[architecture/event bus|event bus]] purpose in godot design is to decouple nodes that far apart from each other.

This avoids **signal-bubbling chains** across multiple ancestors; either let the **parent mediator** handle it locally or publish once on the **bus**. This is a common pitfall called out in guides.

Prefer **request -> decision/execution -> outcome (success or not)** events on the bus (e.g., card_move_requested, then card_moved/card_move_denied). It keeps leafs ignorant and prevents sibling coupling.