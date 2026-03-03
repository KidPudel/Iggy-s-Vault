# Odin Foreign System

The mechanism in Odin for interfacing with external C libraries via `foreign import` and `foreign` blocks.

## What it does

`foreign import` links an external library into the Odin program under a name. Entities from that library are then declared inside a `foreign` block using that name. Foreign procedure declarations require `---` at the end to mark them as bodyless procedure literals (not procedure types).

Foreign procedures use the `cdecl` calling convention by default. Attributes can override the calling convention or link name per entity.

The `"system:"` prefix tells the linker to search standard system library paths (e.g. `/usr/local/lib`) rather than a project-local file.

## Code

```odin
// Link a system library
foreign import lib "system:SDL2"

// Link a project-local library (Windows)
foreign import kernel32 "system:kernel32.lib"

// Declare imported procedures
foreign lib {
    InitSubSystem :: proc(flags: InitFlags) -> c.int ---
}

// Override calling convention and link name
@(default_calling_convention = "std")
foreign lib {
    @(link_name="InitSubSystem") init_sub_system :: proc(flags: InitFlags) -> c.int ---
}
```

## Sources

- https://odin-lang.org/docs/overview/#foreign-system

## Related

- [[package]]
- [[alias]]
- [[environment variables]]

## Process

- What does `---` signify at the end of a foreign procedure declaration in Odin?
- What is the difference between `foreign import lib "SDL2.lib"` and `foreign import lib "system:SDL2"`?
- How does the `@(link_name=...)` attribute interact with the foreign block's name resolution?
