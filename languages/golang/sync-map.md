# sync.Map

A concurrent key-value map in Go's `sync` package that is safe for use by multiple goroutines without additional locking.

## What it does

`sync.Map` is optimized for two access patterns: when entries are written once and read many times, or when multiple goroutines read, write, and overwrite entries for disjoint sets of keys. It stores and loads values as `any`.

Operations: `Store`, `Load`, `LoadOrStore`, `Delete`, `Range`.

## Code

```go
var m sync.Map

m.Store("key", 42)

val, ok := m.Load("key")
if ok {
    fmt.Println(val.(int)) // 42
}

m.Range(func(k, v any) bool {
    fmt.Println(k, v)
    return true // return false to stop
})
```

## Sources

- https://pkg.go.dev/sync#Map

## Related

- [[go-map]]
- [[sync-pool]]
- [[goroutine]]

## Process

- How does sync.Map achieve concurrent safety without a traditional mutex on every operation?
- What does `Range` guarantee about consistency when concurrent modifications happen during iteration?
- When would a regular map with an explicit `sync.RWMutex` be preferable over `sync.Map`?
