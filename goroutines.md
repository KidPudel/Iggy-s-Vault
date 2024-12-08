Actually goroutine is not a lightweight thread, it is an abstraction of a function that is being launched asynchronously and smartly managed by [[runtime]] and controlled by [[golang scheduler]].
[[why goroutines are lightweight]].

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