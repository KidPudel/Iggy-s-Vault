Pooling is the way to maintain a 'pool' of opened [[database-connection]]s, that can be used from one database operation to another, *but* there is the cost of maintaining opened connections.

Creating a pool will also create a 'pool' of connections (lets say 10 connections), so instead of creating connections each time, they are all created once at the creation of pool, 
then we just lease it from a pool, and after return it back.

If all connections are in use, then new requests will have to wait in line

[short and clear explanation](https://www.youtube.com/watch?v=YM1I-uf_k-o)

#database 