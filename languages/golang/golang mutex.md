# sync.Mutex

Mutual exclusion lock. Allows only one goroutine to access a section at a time.

https://pkg.go.dev/sync#Mutex

```go
type SafeCounter struct {
	counter int
	mutex   sync.Mutex
}

func (c *SafeCounter) Inc() {
	c.mutex.Lock()
	defer c.mutex.Unlock()
	c.counter++
}
```
