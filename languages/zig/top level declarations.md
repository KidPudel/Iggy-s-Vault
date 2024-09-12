Top level declarations (global variables) are ***order-independent*** and ***lazily analyzed***, initialization is ***evaluated at compile time*** [[comptime]]

```rust
pub fn main() !void {
    std.debug.print("{d} and {d}", .{a, b});
}

const a = add(2, 3);
var b = add(5, a);



fn add(x: i32, y: i32) i32 {
    return x + y;
}

const std = @import("std");
```

but for the sake of readability, we could do like this;

```rust
const std = @import("std");

const a = add(2, 3);
var b = add(5, a);

fn add(x: i32, y: i32) i32 {
    return x + y;
}

pub fn main() !void {
    std.debug.print("{d} and {d}", .{a, b});
}
```

Note that since `b` is a variable and not a constant, it is evaluated at run-time, rather than the compile time