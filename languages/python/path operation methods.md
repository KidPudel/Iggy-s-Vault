When declaring path operation functions with `def`, it is run in external thread-pool, that is then awaited, instead of being called directly, as it would block the server.

If we are coming from other async framework, where we are using simple `def` to define trivial-compute only operations for tiny performance, in FastAPI it is the opposite
In this case it would be better to use `async def` *unless* your path operations function running code that performs blocking IO


# conclusion
  
that specifically in fastAPI
1. In simple path operation methods, we should use `async def`, because it is running as usual AND not blocking server

2. we should you `async def` *if we perform some IO operations that **inherently asynchronous*** (like async HTTP client library)

3. We should use synchronous `def` path operation method *if we are executing some heavy blocking IO that is **NOT** asynchronous*, because in FastAPI def is run in external treadpool to not block the server

> *NOTE:* if we are using some our custom utility helper function (like for example to follow KISS and S from SOLID), we can use synchronous`def` function (not path operation function) without executing it in a external treadpool, in the body of asynchronous path operation method, since this sync function is in async, and will not block the server