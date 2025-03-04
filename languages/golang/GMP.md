GMP Golang's concurrency and parallelism model
- G - [[goroutines]]
- M - OS threads are used to execute [[goroutines]]. Go runtime uses a pool of OS threads. The number can be controlled by `GOMAXPROCS`
- P - pipelines or processor "slots" available in Go runtime. They are exactly what allows [[golang scheduler]] to distribute the work. The number of P usually set to CPU cores, but can be adjusted. Each P is associated with local goroutine queue to be executed and each P is associated with M (eah to individual or multiple to one)
