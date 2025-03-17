Remote Dictionary Server, which is used for in-memory data storage, used as a [[non-relational database]], cache and message broker

# Usages:
- **Caching**: for quick access, for frequently accessed data
- **Session** management
- **Broker**: Message and task (great for rapid transport of small messages)


# Use
```zsh
# start server
redis-server

# open cli
redis-cli

# or choose connection
redis-cli -h localhost -p 6379
```

# Strings
```
SET key1 "hello"
GET key1 # returns "hello"
DEL key1 # removes "hello"
EXISTS key1 # 1 if true else 0
APPEND key1 " Welcome!" # hello Welcome!

```

# Numeric
- `SET visits 10`
- `INCR visits # 11`
- `DECR visits # 10`

# Lists
- `LPUSH list1 "first" "second" "third" # adds to the head`
- `RPUSH list1 "first" "second" "third" # adds to the tail`
- `LRANGE users 0 -1 # Shows all list items`
- `LREM users "first`
# Hashes: map, collection of key value pairs
- `HSET user:3 name "alice" status "in chains" # assigns to element N`
- `HGET hash:3 status`
- `HGETALL hash:3
- `HLEN hash`
# Unsorted Sets [[mathematical notion of a set]]
- `SADD tags "member1" "member2"`
- `SMEMBERS tags # displays all members`
- `SISMEMBER tags "member3"`
- `SREM tags "member2"`
- `SUNION tags1 tags2 # combines two`
- `SINTER tags1 tags2 # finds common`
- `SDIFF tags1 tags2 # elements in tags1 not in tags2`
# Sorted sets, each element has a score
- `ZADD leaderboard 1 "member1" 2 "member2`
- `ZRANGE leaderboard 0 -1`
- `ZRANGE leaderboard 0 -1 WITHSCORES # ascending (default)`
- `ZREVRANGE leaderboard 0 2 # Top 3 players in descending orer`
- `ZRANGEBYSCORE leaderboard 100 200 # gets element by score`
- `ZRANK leaderboard "Bob" # get rank of an element`
- `ZINCRBY leaderboard 50 "Alice" # Increment score, which changes the order`
- `ZCARD leaderboard # count number of elements`

# Expiration
```zsh
EXPIRE key1 60 # expires in 60 seconds
PEXPIRE key1 60 # expires in 60 milliseconds
SETEXP key2 30 "john" # set value and expiration
PERSIST key1 # remove expiration
TTL key1 # Time remaining in seconds 
PTTL key1 # Time remaining in milliseconds
```
## General
```zsh
PING # check for answer PONG

# switch to default db
SELECT 1

# list all keys in db
KEYS * 

# fubd jets natcgubg a pattern
KEYS user* # all keys starting with user
KEYS *:email # all keys ending with :email

TYPE mykey # check type of the key

DBSIZE # get the number of keys
FLUSHDB # delete all keys in db
FLUSHALL # delete all keys in all databases
```

