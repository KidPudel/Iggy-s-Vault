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



The main point of FSM is the delegation of the state related logic to the states, and orchestrate them with state machine. That's all


Awareness:
- FSM is a coordinator of transitions between states
- State is an executor, it's a logical component that is isolated to that state, it implements its own behavior. It just minds its own business, otherwise it delegates to the other state via FSM.

Meaning:
- fsm is handling coordination of states.
- states which are independent state executors that owner has delegate to them
- owner is a freed from state concerns, and now can handles indpendent stuff or global/common

State transition could be done centralized, and can be decentralized.

It all is general common practices, not strict rules. It all depends and flexible, it is all just tools that you can utilize as you need it.
