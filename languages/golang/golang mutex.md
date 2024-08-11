If we want to restrict an access of a value, between goroutine, this is called - _**mutual exclusion**_.

Stand Go library has a data structure for that! `sync.Mutex`!
With Mutex, we can:
1. `Lock`
2. `Unlock`

> **HOW IT WORKS**: This will allow to synchronize the access, by letting letting other goroutine to `Lock` the mutex, only when it will be `Unlock`ed
```go
package main

import (
	"fmt"
	"sync"
	"time"
)

type SafeCounter struct {
	counter int
	mutex   sync.Mutex
}

func (sfC *SafeCounter) Inc() {
	sfC.mutex.Lock()
	for i := 0; i < 1000000; i++ {
		sfC.counter++
	}
	sfC.mutex.Unlock()
}

func (sfC *SafeCounter) Eql() int {
	sfC.mutex.Lock()
	defer sfC.mutex.Unlock()
	return sfC.counter
}

func main() {
	safeCounter := SafeCounter{}
	go safeCounter.Inc()
	time.After(2 * time.Millisecond)
	fmt.Println(safeCounter.Eql()) // prints 1000000

}

```
