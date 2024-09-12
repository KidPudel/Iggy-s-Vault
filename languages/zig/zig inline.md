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

# Inline loop
This is works differently, it unrolling loop
```rust
pub fn factorial(comptime n: u8) comptime_int {
    var r = 1; // no need to write it as comptime var r = 1
    for (1..(n + 1)) |i| {
        r *= i;
    }
    return r;
}
```

by adding `inline` to the for loop, this will unroll the loop, so that each interaction of the loop is executabed sequentially without branching.
```rust
pub fn factorial(comptime n: u8) comptime_int {
    var r = 1; // no need to write it as comptime var r = 1
    inline for (1..(n + 1)) |i| {
        r *= i;
    }
    return r;
}
```
With this loop is still can do runtime stuff, but branching and the iteration variables are now `comptime`