It is just "tags", which is just a way to create a shortcut to find a node in a SceneTree. They are just hash maps [[Data structures]].
Primary used for finding and referencing nodes in a scene. But an important property is that group is not limited to one instance, but rather the opposite, so it is also primary used for finding and handling sets of nodes/scenes.

You can also manage groups from scripts. The following code adds the node to which you attach the script to the `guards` group as soon as it enters the scene tree.

```gdscript
func _ready():
	add_to_group("guards")
```

Imagine you're creating an infiltration game. When an enemy spots the player, you want all guards and robots to be on alert.

In the fictional example below, we use `SceneTree.call_group()` to alert all enemies that the player was spotted.

```gdscript
func _on_player_spotted():
	get_tree().call_group("guards", "enter_alert_mode")
```
The above code calls the function `enter_alert_mode` on every member of the group `guards`.

```gdscript
func _on_player_spotted():
	get_tree().notify_group("guards", "123")
```
The above code notifies group, which will trigger their `_notification` function

To get the full list of nodes in the `guards` group as an array, you can call [SceneTree.get_nodes_in_group()](https://docs.godotengine.org/en/stable/classes/class_scenetree.html#class-scenetree-method-get-nodes-in-group):
```gdscript
var guards = get_tree().get_nodes_in_group("guards")
```

The [SceneTree](https://docs.godotengine.org/en/stable/classes/class_scenetree.html#class-scenetree) class provides many more useful methods to interact with scenes, their node hierarchy, and groups. It allows you to switch scenes easily or reload them, quit the game or pause and unpause it. It also provides useful signals.

Note that it is very useful, when you nodes are dynamically added/removed at runtime. In any cases where the relations are far.