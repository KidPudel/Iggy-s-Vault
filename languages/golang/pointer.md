# Pointer

Like C pointers but no arithmetic. `*T` holds the address of a T. Zero value is `nil`.

```go
a := 1
p := &a  // address of a
*p = 10  // dereference — mutates a
```

**Value receiver**: function gets a copy — mutations don't affect caller.
**Pointer receiver**: function gets the address — mutations affect caller.

Pass pointers to share or mutate. Pass values to isolate.

https://go.dev/tour/moretypes/1
https://go.dev/doc/effective_go#pointers_vs_values
