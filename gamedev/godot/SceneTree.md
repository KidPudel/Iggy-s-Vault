SceneTree is a singleton object that is a composer and manager of nodes. It manages hierarchy of nodes in a scene (and scenes)
The whole actual *scene tree* can be paused. 

Nodes can be added, fetched and removed. 
Scenes can be loaded, switched and reloaded.

"game" is the main scene
"level_01" is the scene that is added as a child to the "game"
"coin" is the scene that lives in "level_01"
so the end result is
- "Game"
	- "level_01"
		- "coin"

`get_tree().root`: will give you [[viewport in godot]] (get_viewport)
`get_tree().current_scene`: will give you the main node right bellow the root
NOTE: things like [[Unique node]] syntax is not supported, since it will be looked only from its own original root (PackedScene)