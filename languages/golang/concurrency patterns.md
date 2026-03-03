# Go concurrency patterns

Use this note as a reminder, not a full tutorial.

- Main patterns: `pipeline`, `fan-out/fan-in`, `worker pool`, `cancellation with context`.
- Core safety rule: every goroutine must have a clear stop path (close channel or cancel context).

Minimal shape:

```go
ctx, cancel := context.WithCancel(context.Background())
defer cancel()
```

Official references:

- https://go.dev/blog/pipelines
- https://pkg.go.dev/context
- https://go.dev/doc/effective_go#concurrency

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
