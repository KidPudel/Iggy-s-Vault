# Publish/Subscribe (Pub/Sub)
This pattern allows multiple [[consumer]]s in **different [[consumer group]]s** to subscribe to the [[topic]] read from the [[message broker]]

[[event bus]]

# Point-to-Point
Ensures that one [[consumer]] process each message
[[Kafka]] guarantees that each [[partition in kafka]] is read **only by one [[consumer]]**

So in this pattern we have only one [[consumer group]]


# Event streaming
- [[producer]] send a **continuous events** to a [[topic]]
- [[consumer]]s process events as they arrive

# Request-Reply
Request response pattern that allows:
- sending the message to **request [[topic]]**
- listening for response back via **response [[topic]]**
This allows for synchronous response, but with scalability of [[Kafka]]


# Competing Consumers
in some way it is like in worker pool [[concurrency patterns]] , where workers "compete" to handle the job
This pattern allows to distribute the messages across multiple [[consumer]]s within the same [[consumer group]], this will split the workload.
Meaning this pattern is for load balancing the workflow. ([[load balancer]])