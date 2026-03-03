# Go Concurrency Patterns

Patterns: `pipeline`, `fan-out/fan-in`, `worker pool`, `cancellation with context`.

Core rule: every goroutine must have a clear exit path — close a channel or cancel a context.

```go
ctx, cancel := context.WithCancel(context.Background())
defer cancel()
```

Each pipeline stage: receive from upstream via `<-chan`, send downstream via `chan<-`, respect `ctx.Done()`:
```go
func stage(ctx context.Context, in <-chan int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for v := range in {
            select {
            case out <- process(v):
            case <-ctx.Done():
                return
            }
        }
    }()
    return out
}
```

Fan-in (merge): one goroutine per input, WaitGroup closes output when all done.

https://go.dev/blog/pipelines
https://go.dev/doc/effective_go#concurrency
