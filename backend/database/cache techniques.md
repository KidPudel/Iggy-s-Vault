redis like cache, db like 

# Read oriented
## Cache aside
First check in cache, if no, then read from DB
Usecase: Common for read-heavy applications

## Read-through (custom)
The cache is responsible for loading data from DB if it misses in cache
Usecase: Automation of miss handing

## Query caching
Cache results of queries in Redis
Usecase: optimize frequently executed queries


# Write oriented
## Write-through
CPU writes to the cache and DB in a single transaction
This is a safe option

## Write-back (Write-behind)
Write to the cache, and then cache asynchronously will write to DB
But it fail and data will be lost

# Cache Invalidation
Clear cache after changing DB

---
# Specific to [[redis]]
## Distributed Caching (Redis Cluster)
Scale caching across multiple nodes for availability and scalability ([[CAP]])

## Sharding
[[sharding]]
Divide data across multiple Redis instances, to handle large datasets

## Preloading/Bootstrapping
Load data to redis on startup if it is common to use

## Data compression
Compress data before storing it in redis to save memory
