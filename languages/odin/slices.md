Slices are similar to arrays, but their length is not known at compile time, but it still cannot be changed for the slice, unlike [[dynamic arrays]]
Internally they are data structure that store pointer to the array and length of the slice
```odin
s := []int{1, 2, 3}
```
here the array is created and then created a slice that references it.

# shorthands
```odin
a[2:6]
a[:6]
a[2:]
a[:]
```

when grabbing a chunk of a slice
```odin
a[offset:offset+length]
```