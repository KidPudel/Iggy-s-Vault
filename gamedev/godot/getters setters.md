```gdscript
var direction: DirectionAxis:
	set(value):
		direction = value
		_movement = y_direction if direction == DirectionAxis.Y else x_direction
```