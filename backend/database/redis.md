Remote Dictionary Server, which is used for in-memory data storage, used as a [[non-relational database]], cache and message broker

# Usages:
- **Caching**: for quick access, for frequently accessed data
- **Session** management
- **Broker**: Message and task (great for rapid transport of small messages)


# Use
Start server with `redis-server`
check with `redis-cli ping`

# Types
- Strings
```
SET key1 "hello"
GET key1
DEL key1
EXISTS key1
INCR counter
DECR counter
APPEND key1 " Welcome!"
```

- Lists
	- `LPUSH list1 "first" "second" "third"`
- Hashes: map, collection of key value pairs
	- `HSET user:1 name "alice" status "in chains"`
	- `HGET hash1 field1`
	- `HGETALL hash1
- Sets
	- `SADD set1 "member1" member2`
- Sorted sets
	- `ZADD zset 1 "member1" 2 "member2`

# Commands
## General
- `SELECT 1`: switch to default db
- `DBSIZE`: get the number of keys
- `FLUSHDB`: delete all keys in db
- `FLUSHALL`: delete all keys in all databases
