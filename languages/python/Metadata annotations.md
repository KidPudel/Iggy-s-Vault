Using standard library `typing` `Annotated`, we can supply parameters with additional [[metadata]]
```python
from typing import Annotated


def say_hello(name: Annotated[str, "this is just metadata"]) -> str:
    return f"Hello {name}"
```

`Annotated`, adds context specific [[metadata]]


At the base type annotation is the same:
```python
q: str | None = None
```
and
```python
q: Annotated[str | None] = None
```

But the difference is that, in the 2nd case we can provide specific metadata about its context
```python
@app.get("/get_stuff")
async def get_stuff(stuff_to_get: Annotated[list[str] | None, Query(min_length=3, max_length=50)] = None)
	```
This tells explicitly that we can get list of the *query* parameters

```
http://localhost:8000/items/?q=foo&q=bar
```
