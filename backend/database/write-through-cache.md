# Write-Through Cache

A cache writing pattern where data is written to both the cache and the database synchronously in a single operation.

## What it does

Every write goes to the cache and the backing database before the write is acknowledged as complete. The cache and database are always in sync after each write.

No data is lost on cache failure because the database always holds the latest state. Write latency is higher than write-behind because both stores must confirm before returning.

## Sources

- https://docs.aws.amazon.com/whitepapers/latest/database-caching-strategies-using-redis/caching-patterns.html

## Related

- [[write-behind-cache]]
- [[cache-aside]]
- [[read-through-cache]]
- [[cache-invalidation]]

## Process

- Why does write-through have higher write latency than write-behind?
- What happens if the database write fails after the cache write succeeds in a write-through pattern?
- How does write-through cache affect the consistency guarantee compared to write-behind?
