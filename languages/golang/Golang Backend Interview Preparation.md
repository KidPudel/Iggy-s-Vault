- [x] repeat notes
- [ ] create top questions notes and explain them
- [ ] create snake game
- [ ] practice practice practice


# Top questions
## backend
[[backend interview questions]]

## golang
- [[race condition]]
- [[deadlock]]
- [[mutex]]
- [[sync.Map]]
- [[golang interfaces]]
- [[concurrency in go]]
- [[why goroutines are lightweight]]
- [[golang scheduler]]
- [[Data structures]]
- [[slices under the hood]]
- [[go map under the hood]]
- [[channel axioms]]

## What are the zero values of int, string, slice, array, map

[[golang zero values]]
## What are properties of strings go
[[mutable vs immutable]]
    
## Explain how the goroutine scheduler works and how they relate to thread
[[golang scheduler]]
    
## Explain what is put on the stack vs heap in golang
[[segmented stack in go]]
    
- When and where do you make your own interfaces
    
- How do you handle errors in Go
    
- When and where do you use panic and recover
    
- Describe features of go that help protect data in a concurrent application running in parallel.


## сеть, grpc, api
[[internet-protocol]], [[gRPC]], [[API]], [[HTTP2.0]], [[http methods]]

## нагрузка, rps:
- How to measure and monitor RPS:
    - Use monitoring tools like Prometheus with Grafana for real-time metrics
    - Implement custom middleware to count requests
    - Utilize Golang's `http.Server` metrics or third-party packages like `goji/httpstats`
    - Log and analyze server access logs
    - Use application performance monitoring (APM) tools like New Relic or Datadog
- Strategies to increase RPS capacity:
    - Implement caching (e.g., Redis) to reduce database load
    - Use connection pooling for databases
    - Optimize database queries and indexes
    - Implement load balancing across multiple server instances
    - Use content delivery networks (CDNs) for static assets
    - Implement asynchronous processing for non-critical tasks
    - Utilize Golang's concurrency features (goroutines and channels)
- Understanding the relationship between RPS and system resources:
    - CPU usage typically increases with higher RPS
    - Memory usage may grow due to increased concurrent connections
    - I/O operations (disk, network) can become bottlenecks
    - Database connections may become saturated
    - Network bandwidth can limit maximum achievable RPS
- Techniques for load testing and stress testing:
    - Use tools like Apache JMeter, Gatling, or k6 for simulating high load
    - Implement gradual increase in load to identify breaking points
    - Test different types of requests (GET, POST, etc.) and payload sizes
    - Simulate realistic user behavior patterns
    - Monitor system resources during tests
    - Use Golang's built-in benchmarking tools for specific function performance
- How to optimize code and infrastructure for higher RPS:
    - Profile your Go code to identify bottlenecks (using pprof)
    - Optimize algorithms and data structures
    - Implement efficient error handling and logging
    - Use Golang's standard library efficiently (e.g., sync.Pool for object reuse)
    - Optimize JSON marshaling/unmarshaling (consider libraries like easyjson)
    - Implement circuit breakers for external service calls
    - Use message queues for decoupling and handling spikes in traffic
    - Consider using gRPC for efficient inter-service communication
    - Optimize Docker containers and Kubernetes configurations if used

## git: 
[[git-basics]]

## Горутины, каналы, синхронизация: 
[[concurrency in go]]


## Брокеры сообщений: 
[[message broker]]

## финализаторы, мьютексы: 
[[concurrency in go]]

## дженерики: 
[[golang generics]]
## sync.Pool: 
[[concurrency in go]]

## livecodding: