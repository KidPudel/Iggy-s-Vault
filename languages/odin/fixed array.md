fixed length array with bounds checking that can be disabled at a block level for performance if bounds are known to not exceed
```odin
x := [5]int{1, 2, 3, 4, 5}
y := [?]int{1, 2, 3} // automatically inter its length
```

```odin
#no_bounds_check {
	x[n] = 123 // n could be in our out of range of valid indices
}
```