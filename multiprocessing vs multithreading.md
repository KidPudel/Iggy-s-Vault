Threading uses threads, while multiprocessing uses processes.

Threads share the same memory space
while processes have separate memory (which avoid [[deadlock]] and [[race condition]]), but they are more expensive

Multiprocessing is better for CPU bound tasks, because:
- parallel execution
- overcoming GIL

Threads are better for I/O because of:
- shared memory
- efficient handling of I/O operations
- lighter overhead

But for I/O is better coroutines

[[IO-bound vs CPU-bound processing]]