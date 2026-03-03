# Go Slice

A dynamically-sized view over a contiguous segment of an array, declared with `[]T` syntax.

## What it does

An uninitialized slice is `nil` with length 0. Slices have both a length (number of elements) and a capacity (size of the underlying array from the slice's start). `append` returns a new slice and may allocate a new backing array if capacity is exceeded.

Slices are reference types: assigning a slice shares the same underlying array.

## Code

```go
var scores []int                         // nil slice
scores = append(scores, 1)

slice := []int{1, 2, 3, 4, 5}

// slicing
middle := slice[2:5]  // [3 4 5], high is exclusive
a := slice[:5]        // first 5
b := slice[2:]        // from index 2 to end

// make: specify length and optional capacity
s := make([]int, 3)        // len=3, cap=3
s2 := make([]int, 3, 10)   // len=3, cap=10

// copy
dst := make([]int, len(slice))
copy(dst, slice)

// utility
slices.Equal(s1, s2)
```

## Sources

- https://go.dev/tour/moretypes/7
- https://go.dev/blog/slices-intro

