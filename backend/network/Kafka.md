[[message broker]] that designed for higher level of data load, which makes it event streaming platform with message broker functionality.
This makes it a platform for a *real time data transfer with low latency and high-throughput*

Kafka's primarily focused on pub/sub
# Essence of the Kafka
- **[[message broker]]s**: Kafka **cluster** (group of) stores one or more broker servers to receive and store data from producer and then sends it to the data consumer. Each brokers manages storage and replication partition
- **[[topic]]s**: Topics in Kafka are partitioned logs(journals), meaning each topic is divided into one or more partitions
- **partitions**: Sequential, ordered immutable set of records (messages, events) within the [[topic]], each partition is *append-only* log, and records are assigned in unique, sequential identifier, called **offset**
So it's not traditional message queue

![[Pasted image 20240407124019.png]]
