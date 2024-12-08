Channel is a global typed conduct (pipe, tube) or [[message queue]], through which you can send and receive data with the help of channel operator `<-`

> channels kind of like [[errors]] in golang, but for concurrent operations, they are values, so we can pass them, manage them, iterate them, print them, mess around with them.

```go
type chan struct {
	mx sync.mutex
	buffer []T
	readers []Goroutines
	writers []Goroutines
}
```
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


# Buffered channels
---
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

if `bCh` channel would have 1 buffer, then it would except first parameter without blocking

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

we can restrict returning channel by returning receive only channel `<- chan int`

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