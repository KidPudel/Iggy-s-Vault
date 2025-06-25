FSM is a way of coordinating and transitioning between different states

In Godot, implementing a **state machine** (FSM) is a common and clean way to manage character behaviors, enemy AI, UI flows, and more. There are **two main approaches**:
- Node-based FSM (Scene-per-State)
	- Each state is its own Node (often script + scene), and the FSM is a parent node that manages switching between them.
- Script-only FSM (State as Objects / Enums)
	- FSM is implemented via enums or string names, switching logic in a central script.


We have:
classes as nodes
import scenes
just classes as scripts and attached to local nodes like `Node`
