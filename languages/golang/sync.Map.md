`sync.Map` is a concurrent map ([[concurrency in go]]) that provides a safe way to share data between goroutines without having to use explicit synchronization primitives like [[golang mutex]]. Great for use where most of the time reads are needed and fewer writes or updates.
It works by using atomic operations in `Store`, `Load`, `Delete` like synchronization primitives provided by the Go runtime (`sync/atomic` and `sync.Once`)

```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	var m sync.Map

	// Store values concurrently
	wg := sync.WaitGroup{}
	wg.Add(2)

	go func() {
		defer wg.Done()
		m.Store("key1", 123)
	}()

	go func() {
		defer wg.Done()
		m.Store("key2", 456)
	}()

	wg.Wait()

	// Load values concurrently
	wg.Add(2)

	go func() {
		defer wg.Done()
		if value, ok := m.Load("key1"); ok {
			fmt.Println("Value of key1:", value)
		}
	}()

	go func() {
		defer wg.Done()
		if value, ok := m.Load("key2"); ok {
			fmt.Println("Value of key2:", value)
		}
	}()

	wg.Wait()

	// Range over the map
	fmt.Println("All stored items:")
	m.Range(func(key, value interface{}) bool {
		fmt.Printf("Key: %v, Value: %v\n", key, value)
		return true
	})
}

```