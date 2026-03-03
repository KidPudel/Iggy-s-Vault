# recover

Stops a panicking goroutine and returns the panic value. Only works inside a deferred function.

https://go.dev/blog/defer-panic-and-recover

```go
defer func() {
	if r := recover(); r != nil {
		err = fmt.Errorf("panic: %v", r)
	}
}()
```
