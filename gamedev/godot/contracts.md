In godot we can check in a script if the class has this method or not
```gdscript
if body.has_method("on_item_pickup"):
	body.on_item_pickup("pickaxe")
```