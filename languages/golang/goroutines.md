Goroutine are modern version of [[green threads]], it is an abstraction above the function, runtime-managed ***execution context*** (more about that below) that is being launched asynchronously and smartly managed by [[runtime]] and controlled by [[golang scheduler]].
[[why goroutines are lightweight]].

# what is the execution context here?
Since goroutines are manages functions
- Its own [[stack]]
- [[program counter]]
- [[runtime]] [[metadata]], containing information to manage goroutine
	- state: running, waiting, ready
	- which os thread it is currently mapped to


# [[green threads]]?
Yes
1. Execute code concurrently
2. Managed by go [[runtime]], so kernel knows nothing about goroutines
3. scheduling goroutines by [[golang scheduler]]
But more
1. [[runtime]] creates and manages context for each goroutines
2. they are not blocking [[syscalls]]


> NOTE: All goroutines are allocated in [[heap]] and not garbage collected ([[garbage collector]]), so they **must exit on their own**.

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


in 64x is 1gb max
32 250 max