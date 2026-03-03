# defer

Defers a function call to run when the enclosing function returns. Multiple defers stack — last in, first out. Use for cleanup of external resources (files, connections, locks).

https://go.dev/blog/defer-panic-and-recover

```go
f, _ := os.Open(name)
defer f.Close()

mu.Lock()
defer mu.Unlock()
```
