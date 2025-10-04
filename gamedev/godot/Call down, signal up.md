Rule of thumb.
It is fine to call functions on the nodes below (children) the caller (parent),
but not wise versa, **instead** you should use signals of the children and emit it on a children and that way connected listener in a parent will get the information.
If you need to communicate between 2 siblings, then it is a good practice to connect the "wire" between the parent.

Why?
In Godot since [[node in godot]] are components, that are by design is better to kept more isolated (mind your own business). It is a common practice to follow this practice. Which turns direction communication to the lazy and everyone is happy. Otherwise they are too coupled and therefore since the isolated component doesn't know about the parent, if we mix it, then it could break, especially if we change the parent. So don't do it.
But why call down? because down, means to its children, meaning that the children of the node is the part of the node. It is the [[scene]], the composition of the nodes.

# Mental model:
- **Nodes manages its own business,** and acts independently, as isolated component
- Parent nodes that has children should **act as a mediator for managing children cross processes** (when action requires multiple components)
- Child nodes should signal up to the parent, and **don't know a thing about outside world.**
	- If the node wants to communicate with distant parent (from distant nested child), we could utilize the [[gamedev/godot/event bus|event bus]]
- For many **passive listeners** (HUD, SFX), use [[groups]] to broadcast side-effects instead of wiring everything to gameplay signals. get_tree().call_group(...) is built for this.
- In my practice it is good to create the *confirmative* two-way signal connection. Meaning child signals, and then depending on the success or failure child waits for the confirmation to act accordingly.