> In Go pointers like in C, but without arithmetics for safety.

When declaring a pointer `*T` of type `T` we allocate a data in heap, so that we can use it everywhere and any time
So in go, if we don't assign point, its value is `nil`

_pointer to the int value_
```go
var p *int
```

_`&` generates a pointer (returning address, where we can point later)_
```go
a := 1
p = &a
```

_`*` denotes (points to) pointer's underlying value_
```go
fmt.Println(p) // 0x000121 - some address
fmt.Println(*p) // 1
```

_when pointing to (denoting) value that stored in address, we basically accessing its value, so we can change it_
```go
*p = 10
fmt.Println(a) // 10
```
