GMP Golang's concurrency and parallelism model
- G - [[goroutines]]
- M - (Machine?) OS threads are used to execute [[goroutines]]. Go runtime uses a pool of OS threads. The number can be controlled by `GOMAXPROCS`
- P - pipelines or processor "slots" available in Go runtime. They are exactly what allows [[golang scheduler]] to distribute the work. The number of pipelines usually set to CPU cores, but can be adjusted. Each pipeline is associated with local goroutine queue to be executed and each pipeline is associated with M (each to individual or multiple to one). Bridge?
