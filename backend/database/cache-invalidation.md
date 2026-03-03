# Cache Invalidation

The process of removing or expiring cached data when the underlying source data has changed.

## What it does

When a database record is updated or deleted, any cached copy of that data may be stale. Cache invalidation removes or marks the stale entry so the next read fetches fresh data from the source.

Common strategies:
- **TTL (Time-to-Live)**: entries expire automatically after a set duration
- **Event-based**: the application explicitly deletes or updates cache entries on write
- **Write-through**: the cache is updated synchronously on every write, preventing staleness

## Sources

- https://docs.aws.amazon.com/whitepapers/latest/database-caching-strategies-using-redis/caching-patterns.html

## Related

- [[cache-aside]]
- [[write-through-cache]]
- [[write-behind-cache]]
- [[redis]]

## Process

- What is the difference between cache eviction and cache invalidation?
- How does a TTL-based invalidation strategy trade consistency for simplicity?
- What race condition can occur between a cache delete and a concurrent cache read in a cache-aside pattern?
