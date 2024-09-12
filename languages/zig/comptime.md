In Zig we can make some arbitrary code **known and/or be executed at compile time** using `comptime` keyword
So `comptime` means 2 things based on context
1. Marked with `comptime` variable or parameter, must be **known** at compile-time (by the end of it)
2. Marked with `comptime` function call or a block of execution tells the compiler should **execute** it at copile-time.
3. `*` But also there is a `comptime_` types, that are automatically figure out what's the best type to use, because the inputs the functions are known at compile time or `comptime`
# compile time types
> NOTE: Zig threats types as [[first-class citizens]], however they can only used in expressions that are **known** at compile time, 

At the callsite, the value **must be *known* at the compile time**, otherwise it's a compilation error

Compiler time types, means the types that must be known at compile time
```rust
fn max(comptime T: type, a: T, b: T) T {
    if (T == bool) {
        return a == b;
    } else if (a > b) {
        return a;
    } else {
        return b;
    }
}
```
 Compile time parameters

example where we don't know at compile time
```rust
const std = @import("std");

fn max(comptime T: type, a: T, b: T) T {
    if (T == bool) {
        return a == b;
    } else if (a > b) {
        return a;
    } else {
        return b;
    }
}

fn get_type(condition: bool) void {
    const t = if (condition) i32 else u32;
    const max_param = max(t, 3, 5);
    std.debug.print("compile time result {}\n", .{max_param});
}

pub fn main() !void {
    get_type(true);
}

```
so this will not work
```zsh
src/main.zig:21:15: error: value with comptime-only type 'type' depends on runtime control flow
    const t = if (condition) i32 else u32;
              ^~~~~~~~~~~~~~~~~~~~~~~~~~~
src/main.zig:21:19: note: runtime control flow here
    const t = if (condition) i32 else u32;
                  ^~~~~~~~~
```

> **NOTE:** you don't need to explicitly type `comptime` before type, because it is always enforced, so that type parameter is determined at compile time

```rust
pub fn main() !void {
    var prng = std.rand.DefaultPrng.init(blk: {
        var seed: u64 = undefined;
        try std.posix.getrandom(std.mem.asBytes(&seed));
        break :blk seed;
    });
    const rand = prng.random();
    const i = rand.int(u8);

    const t = if (i % 2 == 0) u32 else i32; // error
    std.debug.print("{}", .{t});
}
```

```zsh
src/main.zig:28:15: error: value with comptime-only type 'type' depends on runtime control flow
    const t = if (i % 2 == 0) u32 else i32;
              ^~~~~~~~~~~~~~~~~~~~~~~~~~~~
src/main.zig:28:25: note: runtime control flow here
    const t = if (i % 2 == 0) u32 else i32;
                  ~~~~~~^~~~
```

# Compile-time variables

Marking variable with `comptime`, guarantees to the compiler that every load and store of the variable is **performed** at compile-time, meaning that this variable can be used only in compile time operations. Any violation of this results in a compilation error.
> NOTE: so if we can change variable only at compile time, at run-time this variable is basically a `const`
```rust
comptime var a = 0;
```

And combining it with [[zig inline]], we can write a function which is partially evaluated at compile-time and partially at run-time

```rust
const expect = @import("std").testing.expect;

const CmdFn = struct {
    name: []const u8,
    func: fn(i32) i32,
};

const cmd_fns = [_]CmdFn{
    CmdFn {.name = "one", .func = one},
    CmdFn {.name = "two", .func = two},
    CmdFn {.name = "three", .func = three},
};
fn one(value: i32) i32 { return value + 1; }
fn two(value: i32) i32 { return value + 2; }
fn three(value: i32) i32 { return value + 3; }

fn performFn(comptime prefix_char: u8, start_value: i32) i32 {
    var result: i32 = start_value;
    comptime var i = 0;
    inline while (i < cmd_fns.len) : (i += 1) {
        if (cmd_fns[i].name[0] == prefix_char) {
            result = cmd_fns[i].func(result);
        }
    }
    return result;
}

test "perform fn" {
    try expect(performFn('t', 1) == 6);
    try expect(performFn('o', 0) == 1);
    try expect(performFn('w', 99) == 99);
}
```

this generates following codes
performFn_1
```rust
// From the line:
// expect(performFn('t', 1) == 6);
fn performFn(start_value: i32) i32 {
    var result: i32 = start_value;
    result = two(result);
    result = three(result);
    return result;
}
```

performFn_2

```rust
// From the line:
// expect(performFn('o', 0) == 1);
fn performFn(start_value: i32) i32 {
    var result: i32 = start_value;
    result = one(result);
    return result;
}
```

performFn_3

```rust
// From the line:
// expect(performFn('w', 99) == 99);
fn performFn(start_value: i32) i32 {
    var result: i32 = start_value;
    return result;
}
```


# Compile-time expressions
With `comptime` we can guarantee that expression will be evaluated at compile-time, if this cannot be accomplished, the compiler will emit an error

Within a `comptime` expression:
- All variables and functions are `comptime`
- All `if`, `while`, `for`, `switch` are evaluated at compile-time
- `return` and `try` expression are invalid

We can call the same function at compile time and run time
```rust
fn fibonacci(index: u32) u32 {
	if (index < 2) return index;
	return fibonacci(index - 1) + fibonacci(index - 2);
}

test "fibonacci" {
	// run-time
	try expect(fibonacci(7) == 13);

	// compile-time
	try comptime expect(fibonacci(7) == 13);
}
```

So if we would make it endless, the compiler produces an error which is stack trace from trying to evaluate the function at compile-time, because of the unsigned int!
But if we would make it signed, then compiler would notice that the function at compile-time took more than 1000 branches to execute, this emits an error and gives up.
> **NOTE:** in the future we will be able to change that number

At [[containers]] level, all expressions are **implicitly** `comptime` expressions.
```rust
const first_25_primes = firstNPrimes(25);
const sum_of_first_25_primes = sum(&first_25_primes);
```

# Compile-time types
`comptime_` types, that are automatically figure out what's the best type to use, because the inputs the functions are known at compile time or `comptime`
```rust
fn square(comptime x: comptime_int) comptime_int {
	return x * x;
}

test "square" {
	try std.testing.expectEqual(9, square(3));
}
```



# Why not to use `comptime` for everything?
1. `comptime` *pollutes* the code it touches. Meaning that everything it uses would also need to be `comptime`.
2. You can't use run-time variables