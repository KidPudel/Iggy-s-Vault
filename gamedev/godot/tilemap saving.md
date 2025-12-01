To save tile map correctly make sure to take into account the offset of when the tiles begin
```python
# data to save
var ground_pattern: TileMapPatten = ground_layer.get_pattern(ground_layer.get_used_cells())

# calculate offset
var min_pos: Vector2i = used_cells[0]
for cell in used_cells:
	min_pos.x = min(min_pos.x, cell.x)
	min_pos.y = min(min_pos.y, cell.y)
	_ground_pattern_offset = min_pos


# data to load
ground_layer.set_pattern(SystemManager.farm_system.get_ground_pattern_offset(), ground_pattern)

```