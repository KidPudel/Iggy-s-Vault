> 	Intermediate layer that allows two sides to communicate with each other through messaging protocol, by ***asynchronously***  sending messages to not wait for the response, and just hit the  and maintaining a message queue, as well as following some rules on how to send message

**Intermediate layer** of software that allows two sides to communicate with each other by *translating messages between format messaging protocols*, this allows them "talk" with each other even on different technologies (languages) ***asynchronously** to not wait the consumer to receive the message* and *be sure that message will be delivered* maintaining a [[message queue]]

It is responsible for routing from sender to receiver as well as formatting and filtering messages based on the rules.

# models
## point-to-point messaging
distribution pattern that utilizes message queues with one-to-one relationship between sender and receiver. Each message is *sent to only one receiver by only one sender and must be acted upon only once*.
Use case: is for payment processing, because we need to accomplish financial transaction.

## public/subscribe messaging (pub/sub)
The producer of messages adds a message to the [[topic]] and consumers subscribes to to topics from which they want to receive messages. One-to-many relationship, so all messages are published to a [[topic]] are distributed to all subscribers.
Use case: Updating landing times of an airline, driver position on the map

# message broker vs api
## [[API]]
It is best used for synchronous request/reply. API is designed for immediate response, so if the client is down, then the sending service will be blocked while it awaits the reply

## [[message broker]]
Message brokers enable asynchronous communications, so it doesn't wait for reply from receiver and it can also keep track of consumer state. This improves fault tolerance and resiliency.


# message broker vs event streaming platform
Event streaming is only allows for pub/sub messaging patter. 
Event streaming designed for use with high-volume of messaging, so it is readily scalable.
They are capable of ordering streams of messages into categories called [[topic]]s and store them for predefined amount of time.
But unlike message brokers, event streams cannot guarantee the message delivery or track which consumer get the message.

They are more scalable than message brokers, but offer a fewer features for fault tolerance (*like message resending*), as well as more limited message routing and queueing capabilities.


# message broker vs [[ESB]]
[[ESB]] is challenging to integrate, difficult to maintain, difficult to troubleshoot, not easy to scale.
Message brokers are "lightweight" alternative to [[ESB]], which provides similar functionality - *a mechanism for inter-service communication* - with more simplicity and lower cost.


# use cases
- **Financial transactions and payment processing:** since using point-to-point allows you to get the message once, and be sure that it will be delivered to the person who must receive it.
- **E-commerce order processing and fulfillment:** for fault tolerance 
- **Protecting highly sensitive data at rest and in transit:**
- **Event streaming use cases like taxi position**: with pub/sub

# examples of message brokers
- RabbitMQ - classic message broker
- [[Kafka]] - message broker that designed for higher amount of data load (better throughput), which makes it event streaming platform with message broker functionality 