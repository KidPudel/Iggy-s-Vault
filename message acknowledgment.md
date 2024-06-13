Message acknowledgment is the mechanism in [[RabbitMQ]] [[message broker]], to mark that message has been received and to delete it from the queue.

But this could be dangerous, if consumer doing a long running task to process a message after receiving, and if program stops, the message is lost and no from the queue and from everywhere

By default manual acknowledgement is turned on, but in example with `auto_ack=True`, manual acknowledgement is turned off.

```python
def callback(ch, method, properties, body):
	print("start")
	time.sleep(body.count(b'.')) # as byte
	print("end")
	ch.basic_ack(delivery_tag=method.delivery_tag)
```

> NOTE: acknowledgement must be send in the same channel that received a message/delivery

If you forget to acknowledge with `basic_ack`, after the client reconnect, the message will be redelivered, which may look like a random message, but RabbitMQ will eat more and more memory as it won't be able to release any "unacked" messages

But we can debug it with
```bash
sudo rabbitmqctl list_queues <name> messages_ready messages_acknowledged
```