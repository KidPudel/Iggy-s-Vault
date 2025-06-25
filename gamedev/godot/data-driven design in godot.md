Case: You are picking up an axe from the world (some Node3D, with area and script on picking up logic), obviously we don't want to collect the same object, but rather a **data representation** of the object, that doesn't care about anything, except for data it has itself.
This is a perfect case for [[data-driven design]] to be introduced in Godot.

In Godot to do so, we can simply create a data model, which could be a literal data structure, that acts as a database stored in some basic script/class
```gdscript
class_name Items

const Database = {
	"pickaxe": {
		"name": "Pickaxe"
		"scene": "res://path/to/pickaxe.glb"
	},
	"sword": {
		"name": "Sword"
		"scene": "res://path/to/sword.glb"
	}
}
```

But this approach has several crucial problems:
1. We can easily create a typo in node that uses this like type"swort", which will be a runtime error
2. We can simply move scenes to other places, and the same error will happen
3. Maintaining 100 objects is still hard taking into account 1st and 2nd problems.

We can utilize godot specific feature called [[resource]], which is a data class/structure, which allows for modeling a [[static]] data.
```gdscript
class_name Item extends Resource

@export var name: String
@export var scene: PackedScene # godot will automatically fix the path if we move it somewhere
```

Which then allows as to create specific resources based on our new `Item` type, like `Pickaxe`, `Sword`, etc. and now we can securely reference concrete setup resources.