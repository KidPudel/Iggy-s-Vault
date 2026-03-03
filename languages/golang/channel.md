# Channel

Typed pipe for sending and receiving values between goroutines with `<-`. Channels are values — pass them, store them, range over them. Same idea as errors being values.

https://go.dev/ref/spec#Channel_types
https://go.dev/doc/effective_go#channels

Under the hood:
```go
type chan struct {
	mx      sync.Mutex
	buffer  []T
	readers []Goroutine
	writers []Goroutine
}
```

**Blocking**: unbuffered send blocks until someone receives, and vice versa. This is how you synchronize.

**Buffered**: `make(chan int, n)` — send doesn't block until buffer is full; receive doesn't block until buffer is empty.

**Directional**: `chan<- int` send-only, `<-chan int` receive-only. Use when passing to functions.

**Close**: only the sender closes. Sending to a closed channel panics. `for v := range ch` exits when channel is closed.

**Select**: wait on multiple channel ops — whichever is ready first runs. `default` makes it non-blocking.

Nil channel axioms:
- send to nil → blocks forever
- receive from nil → blocks forever
- send to closed → panic
- receive from closed → returns zero value immediately
