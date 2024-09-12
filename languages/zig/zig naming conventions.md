# Types and namespaces
If `x` is a type then `x` should be `TitleCase`

# Functions and Types
If `x` is callable, and `x`'s return type is `type`, then `x` should be `TitleCase`, otherwise `camelCase`
```rust
fn Point(comptime T: type) type {
	return struct {
		x: T,
		y: T,
	};
}

// struct types
const IntPoint = Point(i32);

pub fn main() void {
	const p = IntPoint{
		.x = 10,
		.y = 12,
	};

	const p2 = Point(u32){
		.x = 12,
		.y = 20,
	};
}
```

# Other
`snake_case`