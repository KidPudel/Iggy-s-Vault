# Concurrency in Go

Fan-out at every I/O boundary in a high-load service. Goroutines are cheap — use them.

**WaitGroup** — wait for a group to finish. Add *before* launching, not inside:
```go
var wg sync.WaitGroup
for i := range work {
    wg.Add(1)
    go func() {
        defer wg.Done()
        doWork()
    }()
}
wg.Wait()
```

`wg.Add` must happen before `go` — `Wait()` can be reached before the goroutine even starts.

See: goroutines.md · channel.md · concurrency patterns.md · golang mutex.md · sync-map.md · sync-pool.md

https://pkg.go.dev/sync
