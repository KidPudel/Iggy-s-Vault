Odin is manual memory management based, so it has no runtime.
To aid with memory management, Odin has huge support for custom allocators, and especially through the implicit [[context]]

Build in types like [[dynamic arrays]] and maps both contain a custom allocator.
This allocator could be manually set or the allocator from the current [[context]] is used instead.

```odin
ptr := new(int)
```
is equivalent to this:
```odin
ptr := new(int, context.allocator)
```

[[context]] stores 2 different forms of allocators that could be changed:
- `context.allocator` - is for "general" allocations, for the subsystem it is used within, by default is an os [[heap]] allocator.
- `context.temp_allocator` - is for temporary and short lived allocations, which are to be freed once per cycle/frame/etc. assigned to a [[scratch allocator]] and `free_all(context.temp_allocator)` must be called to clear the contents of the temporary allocator's internal [[arena]].

> `package mem` also has those [[procedures]], but with enforced allocator errors


- `new` procedure takes type and returns the pointer to the type given
```odin
i_ptr = new(int)
i_ptr^ = 123
x: int = i_ptr^
```

- `new_clone` - allocates a close of the value passed in 
```odin
i := 123
ptr := new_clone(i)
assert(ptr^ == i)
```

- `make` - allocates memory for a **backing data structure** of slice, [[dynamic arrays]], maps
```odin
people := make([dynamic]int, 0, 65)
```

- `free` - frees the memory at the pointer given
- `free_all` - frees all the memory of the context's allocator

- `delete` - deletes the **backing memory** of a value allocated with `make` or a string that was allocated through an allocator


# Tracking allocator
Allocator that warns you if your program is leaking memory of if it does bad frees. stored in `core:mem`
```odin
when ODIN_DEBUG {
	track: mem.Tracking_Allocator
	mem.tracking_allocator_init(&track, context.allocator)
	context.allocator = mem.tracking_allocator(&track)
	
	defer {
		if len(track.allocation_map) > 0 { 
			fmt.eprintf("=== %v allocations not freed: ===\n", len(track.allocation_map))
			for _, entry in track.allocation_map {
				fmt.eprintf("- %v bytes @ %v\n", entry.size, entry.location) 
			} 
		} 
		if len(track.bad_free_array) > 0 {
			fmt.eprintf("=== %v incorrect frees: ===\n", len(track.bad_free_array))
			for entry in track.bad_free_array { 
				fmt.eprintf("- %p @ %v\n", entry.memory, entry.location) 
			}
		}
		mem.tracking_allocator_destroy(&track)
	}
}

```

[[when statement]]