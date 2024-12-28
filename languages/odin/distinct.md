`distinct` type allows for creation of a new type with the same underlying semantics
```odin
My_Int :: distinct int
assert(My_Int != int)
```

like in golang
```go
type MyInt int
```

