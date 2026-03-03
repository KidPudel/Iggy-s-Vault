# Apache Cassandra

A distributed wide-column NoSQL database designed for high availability and horizontal scaling across many nodes.

## What it does

Data is distributed across nodes using consistent hashing on a partition key. Each node is responsible for a range of hash values (token range) in a token ring.

Write path:
1. Write is appended to the commit log (durability)
2. Written to MemTable (in-memory structure)
3. When MemTable fills, flushed to disk as an SSTable (immutable sorted file)
4. SSTables are periodically compacted

Nodes communicate cluster state via the Gossip protocol (peer-to-peer health sharing).

Replication factor determines how many nodes hold copies of each partition. Consistency level controls how many replicas must acknowledge a read/write.

## Sources

- https://cassandra.apache.org/doc/latest/

## Related

- [[non-relational database]]
- [[partitioning]]
- [[database replication]]
- [[Gossip protocol]]
- [[nodes]]
- [[cluster]]

## Process

- What is the relationship between partition key, token, and node assignment?
- What happens to a write if the node receiving it crashes after the commit log but before the MemTable flush?
- How does the consistency level setting interact with the replication factor?
- What does "eventually consistent" mean in Cassandra's context?
