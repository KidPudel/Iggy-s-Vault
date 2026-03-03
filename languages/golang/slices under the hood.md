# Slices Under the Hood

A slice is not an array — it's a header (fat pointer) describing a window into a backing array:

```go
type SliceHeader struct {
    Data uintptr // pointer to backing array
    Len  int     // elements in use
    Cap  int     // total allocated space
}
```

Passing a slice copies the header. The backing array is shared. Mutations through one slice are visible to all others pointing at the same array.

**append()**: if Len < Cap → returns new header, same backing array. If Len == Cap → allocates new array (2x), returns new header.

Footgun — two appends from the same base clobber each other:
```go
s := make([]int, 3, 6)
s = append(s, 1, 2, 3) // len=6, cap=6
a := append(s[:5], 6)  // writes 6 at index 5
b := append(s[:5], 7)  // overwrites index 5 with 7
// a and b point to same backing array — both now show 7
```

**copy()**: never reallocates. Overwrites dst up to `min(len(src), len(dst))`.

https://go.dev/blog/slices-intro
