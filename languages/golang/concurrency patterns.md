https://go.dev/blog/pipelines

Go's concurrency approach makes it easy to build/construct streaming data pipeline that takes the best of both worlds, make efficient use of I/O and multiple CPUs with goroutines
[[IO-bound or CPU-bound processing]]

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
Multiple functions can read from **one** [[channel]], to provide a way to distribute work among a group of workers *potentially* to parallelize CPU use and I/O. This is called ***Fan-out***, it widens/spreads the gates
==**In a high load service, we can "fan-out" at every corner, where there is a [[IO-bound or CPU-bound processing]]**==

```go
	in := gen(5, 34, 3, 12)

	// fan-out
	m1 := sq(in)
	m2 := sq(in)
```


Function can read from multiple inputs (channels), until they are all closed by [[multiplexer]] that **multiplexing input channels into one channel** by reading them, that is closed when all inputs are closed (with synchronization). This is called ***Fan-in***, it is narrows/focuses to the bottle-neck.

with fan-in, we merge/fuse the channels into one by reading them all into one, and then after all are done, we can finish

```go
func merge(ins ...<-chan int) <-chan int {
	outMultiplexer := make(chan int)

	wg := sync.WaitGroup{}
	wg.Add(len(ins))
	// fan-in
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

# Worker pool
---
Allows to distribute tasks across ***fixed*** amount of workers.
It allows to limit the number of goroutines while distributing tasks dynamically among them.
So it is like Fan-out pattern, but controlled
In this pattern:
- workers: goroutines that are waiting for the job
- job queue: channel to which jobs are sent
- results: separate channel

```go
func main() {
	const numWorkers = 3

	jobs := make(chan int, 10)
	results := make(chan int, 10)

	wg := sync.WaitGroup{}

	// spawn workes
	for w := 0; w < numWorkers; w++ {
		wg.Add(1)
		go worker(w, jobs, results, &wg)
	}

	for j := 0; j < 10; j++ {
		// free worker will handle job in a queue
		jobs <- j
	}
	close(jobs)

	wg.Wait()
	close(results)

}
```

```go
// id is just for representation
// free worker will grab first in the queue
func worker(id int, jobs <-chan int, results chan<- int, wg *sync.WaitGroup) {
	defer wg.Done()
	for job := range jobs {
		fmt.Println("worker: ", id)
		fmt.Println("working on: ", job)
		time.Sleep(time.Second)
		fmt.Println("done with: ", job)
		results <- job * 2
	}
}
```


# Stopping short
---
In real life we often don't need to read all values from ingoing channel of earlier stage until it closes
Often we need to just read some values to get going, or we listening for a specific values to exit

If stage fails to consume all incoming values, the goroutine (sender) attempting to send value downstream will block indefinitely
This is a **resource leak**, so we need to cover the case if some downstream stage fail/don't want to receive all inbound values.
One way is to change to outbound [[channel]] to have a buffer, this means it has all space it needs to send all values at once ***without blocking***, which means we don't need a goroutine
```go
func gen(nums ...int) <-chan int {
	out := make(chan int, len(nums))
	for _, num := range nums {
		out <- num
	}
	close(out)
	return out
}
```

but now we need to handle those blocked (completed) goroutines in for example Fan-in merge case.
We don't know exactly how many items we have there, but not reading all the values from channel will still leave the goroutine blocked.

Instead we need to provide an explicit way for downstream stages to indicate that they will stop accepting input.

# Explicit cancellation
---
When some of the nested decides to exit without receiving all the values from out, it must tell the goroutines in the upstream stages to abandon their work.
It does so by sending values on a channel called *done*, or in a modern Golang it is done through [[context]] and it's cancellation method.
Also it could be done by simply closing the cancellation channel [[channel axioms]], which will immediately broadcast the signal to others

```go
func main() {
	// entry point for a whole service
	// many users send to the order request that they need some help (goods)
	// my duty is to help people during the zombie apocalypse
	// read about technology while you integrate it

	ctx, cancel := context.WithCancel(context.Background())
	in := gen(5, 34, 3, 12)

	// fan-out
	md1 := sq(ctx, in)
	md2 := sq(ctx, in)

	// safe pipeline
	mut := merge(ctx, m1, m2)
	fmt.Println(<-mut)
	cancel()

}

```

```go
// top stage
func gen(nums ...int) <-chan int {
	out := make(chan int, len(nums))
	for _, num := range nums {
		out <- num
	}
	close(out)
	return out
}

// next stage
func sq(ctx context.Context, in <-chan int) <-chan int {
	out := make(chan int)
	go func() {
		defer close(out)
		for num := range in {
			select {
			case out <- num * num:
			case <-ctx.Done():
				return
			}
		}
	}()
	return out
}

func merge(ctx context.Context, ins ...<-chan int) <-chan int {
	outMultiplexer := make(chan int)

	wg := sync.WaitGroup{}
	wg.Add(len(ins))
	// eat. one by one
	eat := func(in <-chan int) {
		defer wg.Done()
		for num := range in {
			select {
			case outMultiplexer <- num:
			case <-ctx.Done():
				return
			}
		}
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

now, even that we haven't read all values, we've closed channels, and not wasted resources