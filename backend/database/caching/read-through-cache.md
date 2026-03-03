# Read-Through Cache

A cache reading pattern where the cache itself is responsible for fetching data from the database on a miss.

## What it does

The application reads exclusively from the cache. On a miss, the cache layer queries the database, stores the result, and returns it to the application. The application has no direct knowledge of the database for reads.

The cache acts as the primary data source from the application's perspective.

## Sources

- https://docs.aws.amazon.com/whitepapers/latest/database-caching-strategies-using-redis/caching-patterns.html

## Related

- [[cache-aside]]
- [[write-through-cache]]
- [[cache-invalidation]]
- [[redis]]

## Process

- How does read-through differ from cache-aside in terms of which layer is coupled to the database?
- What happens on the first request to a cold read-through cache with no pre-loaded data?
- How does a read-through cache handle concurrent misses for the same key?
