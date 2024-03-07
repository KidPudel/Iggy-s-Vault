Slices **doesn't store elements like array does**, instead it stores a *header*, describing contiguous section of backing array (info like length, capacity, and [[pointer]] to the array itself).
```go
type SliceHeader struct {
    Data uintptr
    Len  int
    Cap  int
}
```
So when passing slice to the function, you pass a **copy of the header**.
When pointing to the backing array and changing the value, it will affect and *let caller to observe the changes*.
**But** when appending, to the slice, we ***change len and cap locally***, so caller won't be able to observe the changes.

