`@export` tells the godot to save their value (any type) along with the [[resource]] they are attached to, by adding metadata to a script. As well as exposed in the editor to set at the design time.
On scene load the engine restores the value before `_ready()`

This allows for defining a property for the class, that can be preconfigured by hand for each instance

Also we can @export_file which will export string as a path to a file


# Example

```gdscript
@export var enemy_scene: PackedScene
@export var hp: int = 100
@export var camera_target: Node
```
### **In Editor:**
- enemy_scene → lets you pick a .tscn file
- hp → shows an integer field
- camera_target → lets you drag any existing Node from the scene tree into this slot