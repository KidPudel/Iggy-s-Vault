We can overload operators like + - * / using [[magic]] methods a.k.a dunder methods

For example, we need to create a vector type `Vector2` so that it can add, multiply etc
```python
class Vector2:
	x: int
	y: float
	
	def __init__(self, x, y):
		self.x = x
		self.y = y

	def __add__(self, to_add: Vector2) -> Vector2:
		return self.x + to_add.x, self.y + to_add.y
```

| Operator | Magic Method                |
| -------- | --------------------------- |
| **+**    | `__add__`(self, other)      |
| **–**    | `__sub__`(self, other)      |
| *****    | `__mul__`(self, other)      |
| **/**    | `__truediv__`(self, other)  |
| **//**   | `__floordiv__`(self, other) |
| **%**    | `__mod__`(self, other)      |
| ******   | `__pow__`(self, other)      |
| >>       | `__rshift__`(self, other)   |
| <<       | `__lshift__`(self, other)   |
| &        | `__and__`(self, other)      |
| \|       | `__or__`(self, other)       |
| ^        | `__xor__`(self, other)      |
