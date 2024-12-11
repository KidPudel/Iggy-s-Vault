Slices **doesn't store elements like array does**, instead it stores a *header*, describing contiguous section of backing array (info like length, capacity, and [[pointer]] to the array itself).
So it's not an array like in C that is a pointer to the beginning of the array, but rather a [[fat pointer]]
```go
type SliceHeader struct {
    Data uintptr
    Len  int
    Cap  int
}
```
So when passing slice to the function, you pass a **copy of the header**.
When pointing to the backing array and changing the value, it will affect and *let caller to observe the changes*.

**But** when appending `append`, to the slice, and if  cap is exceeded, then slice willwe ***change len and cap locally***, so caller won't be able to observe the changes.

