Sometimes it is necessary to interface with foreign code, such as a C library. In Odin, this is achieved through the `foreign` system. You can "import" a library into the code using the same semantics as a normal import declaration:
```odin
foreign import kernel32 "system:kernel32.lib"

when ODIN_OS == .Windows {
	foreign import lib "SDL2.lib"
} else {
	foreign import lib "system:SDL2"
}
```


to then use exported from the library, we can utilize created by `import` "foreign import name", which can then be used to associate entities within a foreign block.
> NOTE: Due to foreign procedures not having a body declared within this code, you need to append the `---` to distinguish it as a procedure literal without a body and not a procedure type
```
foreign lib {
	InitSubSystem :: proc(flags: InitFlags) -> c.int ---
}
```

Foreign procedure declaration have the **cdecl/c** calling convention by default unless specified otherwise.
The attributes system can be used to change specific properties of entities declared within a block

```odin
@(default_calling_convetion = "std")
foreign lib {
	@(link_name="InitSubSystem") init_sub_system :: proc(flags: InitFlags) -> c.int ---
}
```


# Note about foreign import "system"
When importing something inside a package as usual
```odin
foreign import lib "SDL2.lib"
```
it is assumes you're providing a specific file that is located inside your project's working directory or a path you explicitly provide during compilation

With "system:", prefix you tell Odin to rely on the platform's standard library search mechanism (default system linker)
and it will find `"system:SDL2` for example inside `/usr/lobal/lib`.
So the with "system:" it will look in
1. [[environment variables]]
2. default locations like `/lib` etc
3. explicit flags (look up odin build help)

> Note that you need to actually build the C library first to get dll, dylib, or so
