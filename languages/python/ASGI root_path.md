`root_path` is the mechanism from [[ASGI]] to point out to the [[application server]] the path prefix that is not seen (because of the layer of routing/nesting) to the application.
For example in the proxy with [[NGINX]] or with the sub-applications in [[FastAPI]]


We can add it by adding it to the command line
```
fastapi run main.py --root-path /api/v1
```

or we can do that by adding `root_path` to the application instance

```python
app = FastAPI(root_path="/sub")
```

or in case of sub-applications in mounting
```python
app.mount("/v2", subapi)
```