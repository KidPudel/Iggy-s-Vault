Slices **doesn't store elements like array does**, instead it stores a *header*, describing contiguous section of backing array (info like length, capacity, and [[pointer]] to the array itself).
So it's not an array like in C that is a pointer to the beginning of the array, but rather a [[fat pointer]]
```go
type SliceHeader struct {
    Data uintptr
    Len  int
    Cap  int
}
```
So when passing slice to the function, you pass a **copy of the slice header**.
When pointing to the backing array and changing the value, it will affect and *let caller to observe the changes*.

slices are optimized storage for allocated memory for array
if they can, they will keep the memory the same
`len` is an indicator to where to append, also it is like [[cursor]], as in `append()` section will become lear
`cap` is the edge indicating when to allocate a new memory
# `append()`
if cap doesn't exceeded, `append()` returns *the new struct*, with changed `len`, but **the rest is the same**, meaning the data pointed is the same
if cap hit the limit, then *the new struct* with **new memory** is returned


```go
s := make([]int, 3, 6)
s = append(s, []int{1, 2, 3, 4}...) 
s = append(s, 5)// returning new struct and appends to the same variable, so we are up to date
```
In this example we "keeping in touch" the `s` slice, by updating the "cursor where it points the utilized memory" or `Len`

```go
newS := append(s, 6)
```
In this example we still manipulate the same memory, but `s` slice hasn't updated its `Len` value, so it still shows the same 


lets take more complex example
```go
s := make([]int, 3, 6)
s = append(s, []int{1, 2, 3, 4, 5}...)
newS := append(s, 6) // underlying 1 2 3 4 5 6
newS2 := append(s, 7) // underlying 1 2 3 4 
```
Taking into account what previously was said, we can see:
- they all manipulate the same memory
- appending to new slice, doesn't update the cursor of the original slice
- 2nd and 3rd making `Len++` from the same copy, meaning they override each other
- 2nd updates `Len` from 5 to 6, and placing to 6th element `6`
- 3d updated `Len` from 5 to 6, and overriding 6th element to `7`
- result:
	- underlying array is `1 2 3 4 5 7` 
	- `s = 1 2 3 4 5, len 5, cap 6`
	- `newS = 1 2 3 4 5 7, len 6 cap 6`
	- `newS2 = 1 2 3 4 5 7, len 6 cap 6`,  


```go
mini := s[0:2] // len 2, cap 6
mini = append(mini, 10) // len 3, cap 6
fmt.Println(s) // 1 2 10 4 5
```
since we utilizing the same memory, and pointing to overriding existing memory because `Len` points to the 3rd element and overrides the memory