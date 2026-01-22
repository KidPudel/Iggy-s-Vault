```gdscript
# t_bob: time/progress over time
func _headbob(t_bob: float) -> Vector3:
	var pos: Vector3 = Vector3.ZERO
	pos.y = sin(t_bob * BOB_FREQ) * BOB_AMP
	pos.x = cos(t_bob * BOB_FREQ * 0.5) * BOB_AMP
	return pos

```