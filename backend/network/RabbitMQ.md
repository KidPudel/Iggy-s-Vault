[[message broker]] that accepts and forwards messages, you can think of it like a ***Post Office***.

# Official analogy
> when you put the mail that you want posting in a post box, you can be sure that the letter carrier will eventually deliver the mail to your recipient. In this analogy, RabbitMQ is a post box, a post office, and a letter carrier.
>The major difference between RabbitMQ and the post office is that it doesn't deal with paper, instead it accepts, stores, and forwards binary blobs of data ‒ _messages_.
# Basic terms
RabbitMQ, and messaging in general, uses some jargon.

- _Producing_ means nothing more than sending. A program that sends messages is a _producer_ :
    
- _A queue_ is the name for the post box in RabbitMQ. Although messages flow through RabbitMQ and your applications, they can only be stored inside a _queue_. A _queue_ is only bound by the host's memory & disk limits, it's essentially a large message buffer.
    
    Many _producers_ can send messages that go to one queue, and many _consumers_ can try to receive data from one _queue_.
    
    This is how we represent a queue:
    
- _Consuming_ has a similar meaning to receiving. A _consumer_ is a program that mostly waits to receive messages:
    

> Note that the producer, consumer, and broker do not have to reside on the same host; indeed in most applications they don't. An application can be both a producer and consumer, too.



# Sending

lets go from producer to the consumer.

### Connect
First, we need to connect to the RabbitMQ server (Where RabbitMQ is launched).
This could be our local machine, or a different server ([[IP]] address)
```python
connection = pika.BlockingConnection(pika.ConnectionParameters("localhost"))
```
Now with that connection, we retrieve a communication channel with the connection
```python
channel = connection.channel()
```


### Declare and check queue
Before sending messages to the consumer, we need to make sure that [[message queue]] exists, otherwise RabbitMQ will drop a message
```python
channel.queue_declare(queue="hello")
```
> NOTE: we can check existing queues `sudo rabbitmqctl list_queues`

Now, we are ready to send messages, but we cannot simply send messages directly to the queue. It always needs to go through an [[exchange in messaging model]]

```python
channel.exchange_declare(exchange="logs", exchange_type="fanout")
```


Now we are ready to send our message to the queue

```python
channel.basic_publish(
	exchange="", # default/direct
	routing_key="hello", # specific direct queue
	body="Hello World!",
)
```

Before exiting the program we need to make sure that the network buffers were flushed/cleaned and message has been actually delivered to the RabbitMQ.
We can do it by *gently* closing the connection
```python
connection.close()
```

> NOTE: If message hasn't been sent, (aka basic_publish didn't work), maybe the broker has started without enough free disk memory (by default it needs at least 200 MB) and therefore refusing accept the messages.
> We can check the broker logfile, and setup [[RabbitMQ configuration]] file


# Receiving
Connecting is the same, and as in the sending, we need to **make sure that the queue exists** (*GOOD PRACTICE*)
```python
connection.queue_declare(queue="hello")
```

Receiving messages works by subscribing to the queue with `callback`

```python
channel.basic_consume(
	queue="hello",
	auto_ack=True, # automatic acknowledgment to delete from the queue
	on_message_callback=callback
)
```
[[message acknowledgment]]

```python
def callback(ch, method, properties, body):
	print(body)
```

And now to enter a never ending loop of listening for the event and calling callback when necessary
```python
channel.start_consuming()
```



# Message Delivery Semantics/models
The RabbitMQ supports various messaging patterns/models (look in [[message broker]]) point-to-point, pub/sub