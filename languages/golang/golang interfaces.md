# Interfaces

Use when: you need multiple implementations of the same shape, or you want to wrap/enhance an existing type.

Wrapping: if a function takes an interface, implement that same interface to layer your behavior on top.

Under the hood — 2 fields:
1. Type info (concrete type metadata)
2. Data pointer

This is why `nil` interface ≠ typed `nil`. A `*MyError(nil)` stored in an `error` interface has type info set — the interface is not nil.

```go
var e *MyError = nil
var err error = e  // err != nil — type info is present
```

Type extraction:
```go
v, ok := i.(string)         // type assertion
switch v := i.(type) { ... } // type switch
```

https://go.dev/doc/effective_go#interfaces
https://go.dev/ref/spec#Interface_types
