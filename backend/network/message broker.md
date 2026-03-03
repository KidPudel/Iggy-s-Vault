# Message Broker

An intermediary server that receives messages from producers, routes them, and delivers them to consumers asynchronously.

## What it does

A message broker decouples the sender from the receiver. The sender publishes a message and continues without waiting for the consumer to process it. The broker stores the message in a queue until the consumer retrieves it.

The broker handles:
- Routing messages from sender to the correct receiver
- Storing messages until the consumer is ready
- Message format translation between protocols (in some implementations)
- Delivery guarantees (at-least-once, exactly-once depending on config)

Two common patterns: **point-to-point** (one consumer gets the message) and **pub/sub** (message goes to all subscribers of a topic).

## Sources

- https://www.rabbitmq.com/docs
- https://kafka.apache.org/documentation/

## Related

- [[message queue]]
- [[RabbitMQ]]
- [[Kafka]]
- [[topic]]
- [[producer]]
- [[consumer]]
- [[message patterns]]

## Process

- What happens to a message if the broker crashes before delivery?
- How does a message broker differ from a direct HTTP call in terms of failure handling?
- Where does message ordering become a concern, and which brokers handle it differently?
- What is the difference between a message broker and an event streaming platform like Kafka?
