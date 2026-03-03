The process of copying database into one or more replicas of database in order to:
- defend from data loss, during system failures
- increase traffic (load balancing)
- improve latency

> To remove narrow bottleneck

leader: primary database, where writes and read are happening, also responsible for syncing with other leaders or followers
followers: read-only replicas of the leader

# strategies
- Leader-follower (primary replica): query writes to the leader, and then leader replicates it to the followers
But in case of async,  the leader could go down and the most up to date data is lost
If failover (leader failure) is happened in a simple leader-follower strategy, the replica promoted to be a leader, but **without leader you loose the ability to handler rights**!
- Leader-Leader (Multi-Leader): To *mitigate* leader failure
Introduces lags, because of the need to copy over multiple leaders, but it worth it

- Leaderless replication (most clouds uses)