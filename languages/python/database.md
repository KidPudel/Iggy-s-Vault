# PostgreSQL
[psycopg3](https://www.psycopg.org/) is widely known library to use for PostgreSQL


## one connection many cursors
Connection: is designed to be thread-safe, so we can use one connection in different treads by creating different cursors (every cursor will see changes made in the same session by other cursors)

[[cursor]]: is not designed to be thread-safe. Only one cursor in a connection can be executed at the time.

- less overhead, by managing less connections
- no concurrency, since cursors in one connection, are executing one at the time

We could use it where we don't have a lot of request coming, when we mostly need to only lookup the information, like some profile status, etc.
## [[connection pool]]
- scalability, since it has efficient way of storing connection
- concurrency
- more overhead, since we have to store multiple connections 

We could use it where we have a lot of request coming, like adding to basket, removing, ordering, browsing goods, etc.

```python
dbpool = psycopg_pool.AsyncConnectionPool("dbname= user= host= port=")
```
That pool instance we can use to asynchronously acquire a connection

```python
async with pgpool.connection() as conn:
```
async Connection acquiring

```python
async with conn.cursor() as acursor:
```
asynchronous [[cursor]] that will allow as to fetch the results from database server

```python
await acursor.execute(query="", params={}, prepare=True)
```
This will execute a query. By setting [[preparation]], already executed statement that is optimized, is stored in the server session, further the same queries of the same connection are optimized (even with different parameters)

```python
acursor.fetch[all][one]()
```
[[cursor]] is making request on reading stored result in the database server by traversing and keeping track of the position


```python
conn.commit()
```
to make the changes in the database, if some data is updated


# prevent sql injection
always use placeholders instead of directly putting variable into sql, this will automatically put `ticks` around a variable