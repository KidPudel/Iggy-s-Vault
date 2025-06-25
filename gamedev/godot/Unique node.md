Unique node is noted with `%` is just the same as a regular [[node in godot]], but it is marked as **unique**, which allows to "elevate" the path of the node in a script, which makes the path "constant"
```gd
@onready game_manager: Node = %GameManager
```
Medium [[Memory fragmentation]]
[[cache locality]] is medium (based on scene layout)

NOTE: Unique names (%), are scene scoped, not global. To avoid conflicts when you have multiple instances of the same [[scene]].
Imagine if you had multiple Level01 scenes loaded - global unique names would conflict.

Generally used when you want to interact with exactly one known node.