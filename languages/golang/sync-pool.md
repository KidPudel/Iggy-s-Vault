# sync.Pool

A temporary object pool in Go's `sync` package that stores and reuses allocated objects to reduce garbage collection pressure.

## What it does

`sync.Pool` holds a set of temporary objects. `Get` retrieves an object from the pool or calls `New` to create one if the pool is empty. `Put` returns an object to the pool for reuse.

Objects in the pool may be reclaimed by the GC at any time. The pool does not guarantee that a `Get` will return a previously `Put` object.

## Code

```go
var pool = sync.Pool{
    New: func() any { return new(bytes.Buffer) },
}

buf := pool.Get().(*bytes.Buffer)
buf.Reset()        // clean before use
buf.WriteString("hello")
// use buf...
pool.Put(buf)      // return for reuse
```

## Sources

- https://pkg.go.dev/sync#Pool

