# Go Array

A fixed-size, typed collection of elements in Go, declared with `[n]T` syntax.

## What it does

Arrays in Go have a size determined at compile time — the size is part of the type. An uninitialized array is zero-valued (e.g., `[10]int` is all zeros). Length is retrieved with `len()`.

Arrays are value types: assigning or passing an array copies all its elements.

## Code

```go
var matches [10]int
fmt.Println(matches) // [0 0 0 0 0 0 0 0 0 0]

matches[1] = 100

// array literal
cities := [4]string{"mexico", "new-york", "tokyo", "shanghai"}

// 2D array
var twoDm [2][3]int

fmt.Println(len(matches)) // 10
```

## Sources

- https://go.dev/tour/moretypes/6

