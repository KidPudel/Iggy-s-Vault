# Cache-Aside

A cache reading pattern where the application checks the cache first and queries the database only on a miss.

## What it does

The application is responsible for populating the cache. On a cache miss, the application reads from the database and writes the result into the cache before returning it. On a cache hit, the database is not contacted.

The cache does not load data automatically — the application manages both reads and cache population.

## Code

```python
def get_user(user_id):
    data = cache.get(user_id)
    if data is None:
        data = db.query("SELECT * FROM users WHERE id = ?", user_id)
        cache.set(user_id, data, ttl=300)
    return data
```

## Sources

- https://docs.aws.amazon.com/whitepapers/latest/database-caching-strategies-using-redis/caching-patterns.html

## Related

- [[read-through-cache]]
- [[write-through-cache]]
- [[cache-invalidation]]
- [[redis]]

## Process

- What happens to stale data in a cache-aside pattern when the underlying database record is updated?
- How does cache-aside differ from read-through in terms of who populates the cache on a miss?
- What does a thundering herd problem look like in a cache-aside setup after a full cache flush?
