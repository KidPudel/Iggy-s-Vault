In zig structs are anonymous. Zig infers the type name based on a few rules.
- Struct gets named as a variable in initialization expression of that variable
- If the struct is in the return expression, it gets named after the function it is returning from.

We can return structs in function anonymously

```rust
pub fn Player() type {
	const Self = @This();
	return struct{
		pub fn init() Self {
			return Self{
			};
		}
	};
}
```

This is eliminates the need for having structs explicitly for that cases

In practice this is used more often than
```rust
pub struct Player {
	pub fn InitPlayer() Player {
	}
}
```