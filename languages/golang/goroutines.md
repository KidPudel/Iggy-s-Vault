# Goroutines

Runtime-managed execution context — abstraction over a function that runs concurrently. The Go runtime (not the kernel) owns scheduling via GMP.

Each goroutine has:
- Own stack (starts ~2–8 KB, grows/shrinks automatically)
- Program counter
- Runtime metadata: state (running/waiting/ready), current OS thread

```go
go f(x) // f and x evaluated now, execution of f happens in new goroutine
```

**Critical**: goroutines share the same address space. Any shared data requires synchronization (mutex, atomic, channel).

Goroutines are heap-allocated and are **not** garbage collected — they must exit on their own. Leaked goroutines are a common memory leak.

https://go.dev/doc/effective_go#goroutines
https://go.dev/doc/faq#goroutines
