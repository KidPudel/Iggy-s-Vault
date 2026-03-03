# Go Map

An unordered collection of key-value pairs with O(1) average lookup, declared with `map[K]V` syntax.

## What it does

An uninitialized map is `nil` and cannot accept values — it must be initialized with `make`. Map literals can also initialize with values inline.

Two-value assignment `elem, ok := m[key]` checks existence: `ok` is `false` and `elem` is the zero value when the key is absent.

## Code

```go
// initialized with make
m := make(map[string]int)
m["alice"] = 30
m["bob"] = 25

// map literal
people := map[string]int{
    "alice": 30,
    "bob":   25,
}

// insert/update
m["alice"] = 31

// retrieve
age := m["alice"]

// delete
delete(m, "alice")

// existence check
val, ok := m["alice"]
// ok is false if key absent, val is zero value
```

## Sources

- https://go.dev/tour/moretypes/19

