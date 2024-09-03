Instead of null pointers, zig offer ability for optional type
Null references ere the source of many runtime exceptions, and some claim that is the worst mistake in CS

So zig simply doesn't allow for pointers to be null pointers
```c
const foo: *i32 = @ptrFromInt(0x00) // create pointer from an address of the 
_ = foo
```
will result in the following, which prevents runtime errors
```zsh
$ zig test test.zig 
doctest-36933129/test.zig:2:35: error: pointer type '*i32' does not allow address zero
	const foo: *i32 = @ptrFromInt(0x0);
								  ~~~
```

However, any type can be made optional **including pointers**, by prefixing with `?`, this **explicitly tells us about null case**, and **we know about it** and we can handle that cases with `orelse` providing default value
```c
const ptr: ?*i32 = @ptrFromInt(0x00)
std.debug.print("result: {}", .{*ptr orelse 2})
```

or use basic if

```c
const optional_ptr: ?*i32 = @ptrFromInt(0x00)
if (optional_ptr) |ptr| {
	std.debug.print("{}", .{*ptr})
} 
```