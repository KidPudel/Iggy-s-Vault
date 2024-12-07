https://go.dev/blog/pipelines

Go's concurrency approach makes it easy to build/construct streaming data pipeline that takes the best of both worlds, make efficient use of I/O and multiple CPUs with goroutines
[[IO-bound vs CPU-bound processing]]

but what is pipeline? [[pipeline pattern in golang]]


# Pipeline
---
first we define a top stage, that is going to pass downstream the data
```go
func gen(nums ...int) <-chan int {
	out := make(chan int)
	go func() {
		for _, num := range nums {
			out <- num
		}
		close(out)
	}()
	return out
}
```

next stage is going to connected between top and later stages, so it takes data from upstream and passes downstream
```go
func sq(in <-chan int) <-chan int {
	out := make(chan int)
	go func() {
		for num := range in {
			out <- num * num
		}
		close(out)
	}()
	return out
}
```


setting up pipeline
```go
func main() {
	// setup the pipeline and enjoy
	for num := range sq(sq(gen(5, 34))) {
		fmt.Println(num)
	}
}
```


# Fan-out, Fan-in
---
Multiple functions can read from one channel, to provide a way to distribute work among a group of workers *potentially* to parallelize CPU use and I/O. This is called ***Fan-out***, it widens/spreads the gates


```go
	in := gen(5, 34, 3, 12)

	md1 := sq(in)
	md2 := sq(in)

```


Function can read from multiple inputs (channels), until they are all closed by [[multiplexer]] that **multiplexing input channels into one channel** by reading them, that is closed when all inputs are closed (with synchronization). This is called ***Fan-in***, it is narrows/focuses to the bottle-neck.

with fan-in, we merge/fuse the channels into one by reading them all into one, and then after all are done, we can finish

```go
func merge(ins ...<-chan int) <-chan int {
	outMultiplexer := make(chan int)

	wg := sync.WaitGroup{}
	wg.Add(len(ins))
	// eat. one by one
	eat := func(in <-chan int) {
		for num := range in {
			outMultiplexer <- num
		}
		wg.Done()
	}
	for _, ch := range ins {
		go eat(ch)
	}

	// synchronize
	go func() {
		wg.Wait()
		close(outMultiplexer)
	}()

	return outMultiplexer
}
```

These 2 techniques are great fit for extending [[pipeline pattern in golang]]

```go
	for num := range merge(md1, md2) {
		fmt.Println(num)
	}

```


# Cancellation pattern TODO