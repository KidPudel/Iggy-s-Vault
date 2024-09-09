In zig as in [[C inline]] we can inline a function

```rust
pub inline fn square(x: u32) {
	return x * x;
}

test "square" {
	try std.testing.expectEqual(9, square(3));
}
```
And this will work as expected


But this is not only for functions, but also for conditionals and loops.

But this is works differently, it unrolling things..