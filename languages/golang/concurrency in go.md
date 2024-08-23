[]()# Goroutines
> Goroutines - lightweight threads.

Unlike Kotlin's coroutines, which are kind of workers in a thread, **Goroutines are _actual threads_**, but way more lightweight.  

Unlike usual kernel threads, that are managed by operating system, **goroutines are _user-space threads managed by Go runtime_**.

```go
func sleepTrack(start int) {
  for i := 0; i < 9; i++ {
    fmt.Println("Sleeping, which started at ", start)
    time.Sleep(1000)
  }
}
```

```go
go sleepTrack(start: 19)
```
⬆️ starts a new goroutine and it is put to the queue of the [[golang scheduler]] ⬇️
```go
sleepTrack(start: 19)
```

The evaluation of `sleepTrack` and `start` happens is current goroutine,  
While executing happens in a new goroutine.

> **IMPORTANT**: Goroutines ***share the same address space***, so accessing shared data MUST be in synchronized way (to avoid deadlocks and rase conditions), for that Go has useful primitives like mutexes and atomic for protecting shared data accross Goroutines


# Channels
> Channels are typed conduct (pipe, tube), through which you can send and receive data with the help of channel operator `<-`

### send data through channel
```go
func countSum(arr []int, ch chan int) {
  sum := 0
  for i, v := range arr {
    sum += v
  }
  // sending data through conduct
  ch <- sum
}
```
> NOTE: if we want to pass "send-only" channel we declare it as `ch chan<-int`
### receiving data from channel
```go
rightSide := <-ch
```

### Creating channel
```go
ch := make(chan int)
```

### Waiting for the result
By default, sends and receives operations are block execution of their goroutine where they heppened, until the other side is ready.  

**Simple example**: calculating goroutine starts to execute algorithm, we encounter receive operation of channel that is in calculating goroutine, since algorithm is still processing (send operation is not encouintered), receive will block the further execution on its side, unitl we encounter send operation.

**Second example**: calculating goroutine already encountered the result (send operation), but doesn't see that channel is ready to receive (doesn't encountered receive operation), so it will be blocked until, we call to receive a value.
```go

s := []int{7, 2, 8, -9, 4, 0, 1}
ch := make(chan int)

go countSum(s[len(s)/2:], ch)
go countSum(s[:len(s)/2], ch)

// waiting for sending
x, y := <-ch, <-ch

fmt.Println(x, y, x+y) // 17 -4 13
```
Here you may ask: _"How does `x, y := <-ch, <-ch` works, that launching 2 goroutines with one channel, still asigns each variable the **correct part** of the `s` slice?"_.  

After starting both goroutines, we encounter first receive operation (block main function), meanwhile, there are 2 goroutines that are executed, and which ever executes first, will go to the first receiver, and we encounter next receiver and giving it the next _send_ data.


## Buffered channels
We can buffer a channel, by adding a number of buffers as a 2nd parameter in `make` function. Sends won't block until buffer is full, and receives won't block until buffer is empty
```go
bCh := make(chan int, 2)

bCh <- 10
// won't be blocked, bacause buffer is still have 1 space left
bCh <- 5

x := <-bCh // 10
// won't be blocked, because there is still 1 value left
y := <-bCh // 5
```

if `bCh` channel would have 1 buffer, then it would cause a [[deadlock]]

## Closing and Ranges
We must close a channel, to indicate that there is no values will be send. This will finish the process of the program
> NOTE: Only sender (any possible sender, not receiver, meaning we can delegate closing to other goroutine that will **wait** for others) can close channel, otherwise, sending to closed channel, will cause a panic

```go
package main

import "fmt"

func fibonacci(ch chan int) {
	x, y := 0, 1
	for i := 0; i < cap(ch); i++ {
		ch <- x
		x, y = y, x+y
	}
	close(ch)
}

func main() {
	fmt.Println("")
	ch := make(chan int, 10)

	go fibonacci(ch)

	fmt.Println("do some for me")

	for v := range ch { // convinient way to get all from ch
		fmt.Println(v)
	}

	possibleValue, ok := <-ch

	if !ok {
		fmt.Println("Already closed")
	} else {
		fmt.Println(possibleValue)
	}

}

```

## Select
Imagine we wait (or listen) on one of the asynchronious events, asynchroniously, meaning we need channels, and which is going to trigger, the corresponding handler will be executed.  

_And for that, `select` lets goroutine to wait on mutliple operations_.  

Remember, how goroutine will be blocked when we 1. send to the channel, until we encounter a receive. 2. Receive from the channel, until we send to it some value.  

In other word, `select` is mechanism to `select` the triggered (first received value from channel) event.  

```go
func fibonacciListener(onFibonacciReceived, onQuit chan int) {
	x, y := 0, 1
	// listen
	// if fibonacci received, then send next, the same as by default, but with select now we can at the same time listen to quit event
	for {
		select {
			case onFibonacciReceived <- x:
				x, y = y, x+y
			
			case <-onQuit:
				fmt.Println("quit")
				return
		}
	}
}

func main() {
	onFibonacciReceived := make(chan int)
	onQuit := make(chan int)

	go func() {
		for i := 0; i < 10; i++ {
			fmt.Println(<-onFibonacciReceived)
		}
		onQuit <- 1
	}()
	go fibonacciListener(onFibonacciReceived, onQuit)
}
```

### Default selection
`default` case in `select`, is a case that is run, when all other cases are blocked
```go
package main

import (
	"fmt"
	"time"
)

func main() {
	tick := time.Tick(time.Millisecond * 100)
	boom := time.After(time.Millisecond * 500)

	for {
		select {
		case <-tick:
			fmt.Println("tick")
		case <-boom:
			fmt.Println("BOOM")
			return
		default:
			fmt.Println(".")
			time.Sleep(time.Millisecond * 10)
		}
	}
}

```

# Waiting for a group of goroutines
To catch/**wait** for all **goroutines to finish**, use `sync.WaitGroup`.  
It works by counting active goroutines, by using:
	- `Add(n)` and `Done()`
 	- `Wait()`: waits until **all** goroutines are done
> SIDE NOTE: if we want to pass a `wg`, use pointer

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