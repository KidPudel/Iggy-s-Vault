Go's [[garbage collector]] is *concurrent*, so cleaning will take *much*  less time.

Go's GC doesn't work with reference counter, instead it uses **tracing reachable objects GC** strategy, which allows to avoid cyclic problems
Go collects unreachable objects via  [[mark-and-sweep]] algorithm
GC works concurrently which minimizes risks of "**stop the world**", but costs extra power