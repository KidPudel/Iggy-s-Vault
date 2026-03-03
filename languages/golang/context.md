# context.Context

Carries 3 things down the call stack (never up):
1. **Deadlines** — max time before automatic cancellation
2. **Cancellation** — signal goroutines to stop via `ctx.Done()`
3. **Request-scoped values** — auth, trace IDs, per-request data

Keys must be a custom type, not `string` — avoids collisions between packages:
```go
type ctxKey string
ctx := context.WithValue(r.Context(), ctxKey("userID"), id)
val := ctx.Value(ctxKey("userID"))
```

Cancellation pattern:
```go
ctx, cancel := context.WithCancel(context.Background())
defer cancel()

// goroutines respect it:
select {
case out <- result:
case <-ctx.Done():
    return
}
```

https://pkg.go.dev/context
https://go.dev/blog/context
