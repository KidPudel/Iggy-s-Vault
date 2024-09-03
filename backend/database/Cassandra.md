Developed at Facebook [[NoSQL database]] to manage massive amounts of data fast
# Architecture
Nodes and cluster ([[nodes]] and [[cluster]]),
- Node is a single instance of a Cassandra running on a server
- Cluster a collection of nodes

Data distribution
- [[database partition]]: distributes data across nodes using a consistent hashing mechanism, so data is distributed based on partition key
- token ring: each node is responsible for a range of tokens (hash values). **Data is mapped to tokens**, and nodes manage ranges of these tokens

[[database replication]]

Data storage and consistency
- Commit log. Every write operation is first recorded in a commit log for durability. this log is used to recover data in case of node failure
- MemTable: data is first written to in-memory structure MemTable and when full flashes to disk as an SSTable (Sorted String Table)
- SSTable: immutable files stored on disk (periodically merged)

[[Gossip protocol]]
peer-to-peer gossip protocol for node-to node communitcation. This protocol helps nodes share information about the state and health of other nodes in the cluster. Ensuring that all the nodes are aware of changes and overall health of the cluster.