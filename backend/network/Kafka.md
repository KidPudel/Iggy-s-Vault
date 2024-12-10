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