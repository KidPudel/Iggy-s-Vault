# Memory Allocation

Think in lifetimes — allocations per lifetime of a part of the program, not per line.

Go allocates when:
- `var` declarations (stack or heap — escape analysis decides)
- `new(T)` — zero-allocates T, returns `*T`
- `make(T)` — allocates and initializes, returns T (slice/map/chan)
- type conversions: `int`→`string`, `string`→`[]byte`/`[]rune`, value→interface
- string `+` concatenation
- `append` exceeds cap
- map grows past load factor

`new` → pointer to zeroed value
`make` → initialized value (not a pointer)

https://go.dev/doc/faq#stack_or_heap
