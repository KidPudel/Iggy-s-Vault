# Goroutines
---
[[goroutines]]
# Channels
---
[[channel]]
# Patterns
---
[[concurrency patterns]]

## When to use concurrency

In a high load service, we can "fan-out" at every corner, where there is a [[IO-bound or CPU-bound processing]]
and im not talking specifically about fan-out pattern, but rather about concurrency feature like delegating I/O or CPU heavy processes to several "workers" in this case - [[goroutines]]


---
# Waiting for a group of goroutines
To catch/**wait** for all **goroutines to finish**, use `sync.WaitGroup` to easily observe the group.  
It works by counting active goroutines, by using:
	- `Add(n)` and `Done()`
 	- `Wait()`: waits until **all** goroutines are done
> SIDE NOTE: if we want to pass a `wg`, use pointer to affect its changes

> **NOTE**: since the execution is goes so fast, order of execution is **NOT** guaranteed, **_meaning that `wg.Wait()` could be reached faster than `wg.Add(1)`, and execution will be ended by this point._**  
> **THEREFORE**: It is a good practice to add to the group, _BEFORE_ launching goroutine!
```go
func work(id int, wg *sync.WorkGroup) {
	defer wg.Done()
	fmt.Println("started", id)
	time.Afet(time.Second)
	fmt.Println("done", id)
}

func main() {
	var wg sync.WaitGroup
	for i := 0; i < 10; i++ {
		wg.Add(1)
		go work(i, &wg) // func work(id int, wg *sync.WorkGroup)
	}
	wg.Wait() // waits until all goroutines are done
}

// or

func main() {
	var wg sync.WaitGroup

	for i := 0; i < 10; i++ {
		wg.Add(1)
		go func {
			work(i)
			wg.Done()
		}
	}
	wg.Wait() // waits until all goroutines are done
}
```

# Mutex
If we want to restrict an access of a value, between goroutine, this is called - _**mutual exclusion**_.

Standard Go library has a data structure for that! `sync.Mutex`!
With Mutex, we can:
1. `Lock`
2. `Unlock`

> **HOW IT WORKS**: This will allow to synchronize the access, by letting other goroutine to `Lock` the mutex, only when it will be `Unlock`ed
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
	defer sfC.mutex.Unlock()
	for i := 0; i < 1000000; i++ {
		sfC.counter++
	}
}

func (sfC *SafeCounter) Eql() int {
	sfC.mutex.Lock() // waits until unlocked
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


# RWMutex
`RWMutex` is a reader/writer mutual execution lock.
Can be held by **arbitrary/any** number of readers or only a **single** writer
This is the point of RWMutex, compared to regular Mutex, ***multiple readers can lock with `RLock`/`RUnlock`***, but only 1 writer is allowed with `Lock`/`Unlock`

# Pool
Pool in `sync` standard library is a set of temporary objects that may be individually send and retrieved that is safe to use across the goroutines *simultaneously* 
> **NOTE**: Any object can be automatically removed without notification

> **MAIN PURPOSE**: ***cache allocated*** but unused items for later to reuse, therefore ***constraining*** things that are expensive, ***relieving pressure on a garbage collector*** for allocating and deallocating. Build efficient, thread-safe free lists (***but not for all lists, like short-lived objects***)

- `Get() any`: selects the ***arbitrary*** item from the Pool and ***removes*** it from it.
- `Put(x any)`: adds to the pool
- `New()`: optionally specifies a function to generate a value, ***when `Get()` otherwise return `nil`***

# Binary Tree Reading
```go
package main

import (
	"golang.org/x/tour/tree"
	 "fmt"
)

// Walk walks the tree t sending all values
// from the tree to the channel ch.
func Walk(t *tree.Tree, ch chan int) {
	// start printing from left most
	if t.Left != nil {
		Walk(t.Left, ch)
	}
	ch <- t.Value
	if t.Right != nil {
		Walk(t.Right, ch)
	}

}

// Same determines whether the trees
// t1 and t2 contain the same values.
func Same(t1, t2 *tree.Tree) bool {
	firstChannel := make(chan int)
	secondChannel := make(chan int)
	go Walk(t1, firstChannel)
	go Walk(t2, secondChannel)
	
	isSame := true
	
	for i := 0; i < 10; i++ {
		if <-firstChannel != <-secondChannel {
			isSame = false
		}
	}
	
	return isSame
	
}

func main() {
	t := tree.New(3)
	t2 := tree.New(2)
	pipe := make(chan int, 10)
	go Walk(t, pipe)
	
	for v := range pipe {
		// + 1, because we have already read 1 (10 - 1)
		if len(pipe) + 1 == cap(pipe) {
			close(pipe)
		}
		fmt.Println(v)
	}
	
	isSame := Same(t, t2)
	fmt.Println(isSame)
}
```



# fetching api

```go
package main

import (
	"encoding/json"
	"fmt"
	"net/http"
	"strconv"
	"sync"
)

// https://cash.rbc.ru/cash/json/converter_currency_rate/?currency_from={from_currency}&currency_to={to_currency}&source=cbrf&sum={amount}&date=

func convert(currency string, ch chan<- string, wg *sync.WaitGroup) {
	defer wg.Done()

	type Result struct {
		Data struct {
			SumResult float64 `json:"sum_result"`
		} `json:"data"`
	}
	url := fmt.Sprintf("https://cash.rbc.ru/cash/json/converter_currency_rate/?currency_from=%s&currency_to=RUR&source=cbrf&sum=1&date=", currency)

	resp, err := http.Get(url)
	if err != nil {
		fmt.Println("Error fetching data")
		return
	}

	body := resp.Body
	defer body.Close()

	var data Result
	if err := json.NewDecoder(body).Decode(&data); err != nil {
		fmt.Println("Error decoding data")
		return
	}
	ch <- strconv.FormatFloat(data.Data.SumResult, 'f', -1, 64)
}

func main() {
	currencies := []string{"USD", "EUR", "GBP"}
	ch := make(chan string, len(currencies))
	wg := sync.WaitGroup{}

	for _, currency := range currencies {
		fmt.Println(currency)
		wg.Add(1)
		go convert(currency, ch, &wg)
	}

	go func() {
		wg.Wait()
		close(ch)
	}()

	fmt.Println("waiting for ch")
	for result := range ch {
		fmt.Println(result)
	}
}

```