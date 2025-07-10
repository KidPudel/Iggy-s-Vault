Rule of thumb.
It is fine to call functions on the nodes below (children) the caller (parent), but not wise versa, **instead** you should use signals of the children and emit it on a children and that way connected listener in a parent will get the information
If you need to communicate between 2 siblings, then it is a good practice to connect the "wire" between the parent.


	In Godot since [[node in godot]] are components, that are by design is better to kept more isolated (mind your own business). It is a common practice to follow this practice. Which turns direction communication to the lazy and everyone is happy.
But why call down? because down, means to its children, meaning that the children of the node is the part of the node. It is the [[scene]], the composition of the nodes.