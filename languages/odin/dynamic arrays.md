Dynamic arrays are similar to [[slices]] in odin, except that its length is may change during runtime.
They are similar to slices in go. read [[slices under the hood]]

```odin
Raw_Dynamic_Array :: struct {
	data:      rawptr,
	len:       int,
	cap:       int,
	allocator: Allocator,
}
```


```odin
package main

import "core:fmt"
import "core:mem"

main :: proc() {
	s := make([dynamic]int, 0, 4)
	append(&s, ..[]int{1, 2, 3})

	// to replicate go's slice behavior, we need to remember
	// append changes underlying array, and returns an updated new struct

	// create a new struct, with the same content
	s1 := s
	append(&s1, 4)

	s2 := s
	append(&s2, 5) // override

	s3 := s
	append(&s3, 6, 7) // allocates a new underlying array and kills the previous

	fmt.println(s, s1, s2, s3)
}

```
