# Go Generics

Parameterized types and functions in Go, introduced in Go 1.18, using type parameters and constraints.

## What it does

Type parameters are declared in `[T constraint]` syntax. Constraints specify which types `T` can be. `any` allows all types but provides no operations. Interfaces used as constraints define a type set — the set of types that satisfy the constraint.

The `~T` prefix in a constraint includes all types whose underlying type is `T`. Type inference lets the compiler deduce type parameters from function arguments without explicit instantiation.

## Code

```go
// type parameter with constraint
func min[T ~int | ~float64](a, b T) T {
    if a < b { return a }
    return b
}

min(2, 3)         // inferred: T = int
min(2.3, 5.65)    // inferred: T = float64

// interface as type set (constraint)
type Number interface {
    ~int | ~float64 | ~float32
}

func sum[N Number](nums []N) N {
    var total N
    for _, n := range nums { total += n }
    return total
}
```

## Sources

- https://go.dev/doc/faq#generics
- https://go.dev/blog/intro-generics

