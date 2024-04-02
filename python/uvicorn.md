uvicorn is the [[ASGI]] server implementation for the Python


We can run the server by
```bash
uvicorn module_name:instance_of_fastapi_name --reload
```
- `--reload`: make the server restart after code changes (for development )