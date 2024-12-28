All memory allocations together are just a ***lifetime***, this is the best way of thinking about memory management, think of allocations per lifetime of some part.


In Golang memory allocation happening when:
- declare variables
- call `new` function
- call `make` function
- convert
	- `int` -> `string`
	- `string` -> `[]byte` and vice versa
	- `string` -> `[]rune`
	- box values (non-interface) -> interfaces
- concatenate strings with `+`
- convert between strings and byte slices and vice versa
- append to the slice that causing to extend `cap` [[slices under the hood]]
- in map if the underlying array in map is not enough to store the value


# new
`new` function allocates the memory for the type and returns the pointer to that type
```go
p := new(Point) // returns *Pointer
```

# make
`make` function allocates the memory for the type and returns the object itself
```go
s := make([]int, 0, 6) // []int
```
returns the the array with the memory allocated for it