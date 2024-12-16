Kafka is distributed event streaming platform, that is designed to handle real-time data efficiently

It works by [[producer]] sending message (creating a record) to the [[message queue]] and [[consumer]] will read those records with the help of [[message broker]]
Records then identified by sequential number called

The load could become really heavy, and server that handling the [[message queue]] could start struggling to process.
Here comes the [[partition in kafka]], the host of the one or more partition is a [[message broker]]
To which partition to write a record comes the [[partition key]]

So record could be found by [[partition in kafka]] number and [[offset in partion]]

A group of the partitions that store the same type of messages are called [[topic]]
A group of the consumers are called a [[consumer group]]


Kafka has different [[kafka delivery semantics]]

[[message patterns]]


# Getting started
To get started we need to start ZooKeeper or kRaft

To do that we need to install kafka and use their script commands

kRaft
```bash
$ KAFKA_CLUSTER_ID="$(bin/kafka-storage.sh random-uuid)"
```
format logs
```bash
bin/kafka-storage.sh format --standalone -t $KAFKA_CLUSTER_ID -c config/kraft/reconfig-server.properties
```
start kafka server
```bash
bin/kafka-storage.sh format --standalone -t $KAFKA_CLUSTER_ID -c config/kraft/reconfig-server.properties
```


create [[topic]]
```bash
bin/kafka-topics.sh --create --topic quickstart-events --bootstrap-server localhost:9092
```

to see the information on [[topic]]
```bash
bin/kafka-topics.sh --describe --topic quickstart-events --bootstrap-server localhost:9092
```

write events into the [[topic]]
```bash
bin/kafka-console-producer.sh --topic quickstart-events --bootstrap-server localhost:9092
```

read the events
```bash
bin/kafka-console-consumer.sh --topic quickstart-events --from-beginning --bootstrap-server localhost:9092
```

https://github.com/segmentio/kafka-go