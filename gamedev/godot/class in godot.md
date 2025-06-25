**Table of Contents:**
1. [Overview](#overview)
2. [Global class](#Global%20class)
3. [Inheritance](#Inheritance)
4. [Static constructor](#Static%20constructor)
5. [Inner class](#Inner%20class)
6. [Load class to the scene](#Load%20class%20to%20the%20scene)

[Official documentation on classes](https://docs.godotengine.org/en/stable/tutorials/scripting/gdscript/gdscript_basics.html#classes)
# Overview
In Godot, by default all scripts are unnamed [[class]], meaning You don’t need to write something like this explicitly.

But, You can use them only by loading them by their path, and only then you can instantiate it, *since they are "unregistered"*.
```gdscript
# Inherit from 'character.gd'.
extends "res://path/to/character.gd"

# Load character.gd and create a new node instance from it.
var Character = load("res://path/to/character.gd")
var character_node = Character.new()
```

To register a class we can use `class_name` or `class`.



# Global class
By using `class_name`, godot **registers class globally**, meaning they become available to use in other scripts without the need to use `load` or `preload`.

```gdscript
# Saved as a file named 'character.gd'.
class_name Character

var health = 5

func _init():
	pass

func print_health():
	print(health)
```

Now we don't need to use `load`
```gdscript
var player

func _ready():
	player = Character.new()
```

# Inheritance

We can inherit from other classes by using `extends`
> NOTE: If not explicitly defined, class inherits from [RefCounter]([RefCounted](https://docs.godotengine.org/en/stable/classes/class_refcounted.html#class-refcounted)), which provided a way for the [[garbage collector]] to know when it can release the object from memory.

```gdscript
# Inherit/extend a globally available class.
class_name Player extends Node2D

# Inherit/extend a named class file.
extends "somefile.gd"

# Inherit/extend an inner class in another file.
extends "somefile.gd".SomeInnerClass
```

Check inheritance we can using `is`:
```gdscript
if entity is Enemy:
	entity.apply_damage()
```

To call a function in a _super class_ (i.e. one `extend`-ed in your current class), use the `super` keyword:
```gdscript
# idle.gd (inheriting class).
extends "state.gd"

# constructor, must be satisfy it's parent
func _init(e=null, m=null):
	super(e)
	# Do something with 'e'.
	message = m
```

# Static constructor
[[static]] constructor is called automatically, when *the class itself* is loaded, after `static` variables have been initialized.

# Inner class
Inner class is a class that leaves inside the script/global class.
```gdscript
extends CharacterBody2D

enum DirectionAxis {
	X,
	Y
}

class DirectionProperties:
	var axis_key: DirectionAxis,
	var movement_key: Array
	func _init():
		axis_key = DirectionAxis.X,
		movement_key = x_movement	


func _init():
	direction_properties = DirectionProperties.new()
```

# Load class to the scene
If class extends `Node`, it can be loaded to the scene
```gdscript
# spawner.gd
extends Node2D
func _ready():

    var e := Enemy.new()

    e.position = Vector2(100, 100)

    add_child(e)
```


