# Write-Behind Cache

A cache writing pattern where data is written to the cache immediately and flushed to the database asynchronously.

## What it does

The application writes only to the cache, which returns immediately without waiting for the database. The cache queues the write and propagates it to the database in the background.

If the cache fails before the flush completes, queued writes that have not yet reached the database are lost.

Also called write-back.

## Sources

- https://docs.aws.amazon.com/whitepapers/latest/database-caching-strategies-using-redis/caching-patterns.html

## Related

- [[write-through-cache]]
- [[cache-aside]]
- [[cache-invalidation]]
- [[redis]]

## Process

- Under what failure condition is data at risk of loss in a write-behind pattern?
- How does write-behind compare to write-through in terms of write latency and durability?
- What is the relationship between write-behind and the concept of eventual consistency?
