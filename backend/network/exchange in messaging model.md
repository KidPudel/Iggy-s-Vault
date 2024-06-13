Actually, producer quite often doesn't even know if a message will be received to any of the queues. Receiver sends a message to the *exchange*.
Exchange is the simple thing in a [[RabbitMQ]] messaging model, it receives a message from producer and pushes it to a queue. It must know exactly what to do with a message, should it be appended to a particular queue or all of them?
![[Pasted image 20240610132906.png]]

There are a few types of exchange:
- `direct`: goes exactly to specified [[message queue]]
- `topic`
- `headers`
- `fanout`: broadcasts all the messages exchange receives to all of the queues it knows
```python
channel.exchange_declare(exchange="logs", exchange_type="fanout)
```

> NOTE: to list all exchanges on the server run the ever useful `rabbitmqctl`
> `sudo rabbitmqctl list_exchanges`

To create the most basic default (direct) exchange to be able to send messages to the queue, we can can name it with `""` empty string


```python
channel.basic_publish(
	exchange="logs", # fanout exchange
	routing_key="", # since exchange is fanout, we don't need to specify queue
	body=message
)
```