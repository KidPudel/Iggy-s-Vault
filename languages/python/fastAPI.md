Starlette (and **FastAPI**) are based on [AnyIO](https://anyio.readthedocs.io/en/stable/), which makes it compatible with both Python's standard library [asyncio](https://docs.python.org/3/library/asyncio-task.html) and [Trio](https://trio.readthedocs.io/en/stable/).

- **Fast**: Very high performance, on par with **NodeJS** and **Go** (thanks to Starlette and Pydantic). [One of the fastest Python frameworks available](https://fastapi.tiangolo.com/#performance).
- **Fast to code**: Increase the speed to develop features by about 200% to 300%. *
- **Fewer bugs**: Reduce about 40% of human (developer) induced errors. *
- **Intuitive**: Great editor support. Completion everywhere. Less time debugging.
- **Easy**: Designed to be easy to use and learn. Less time reading docs.
- **Short**: Minimize code duplication. Multiple features from each parameter declaration. Fewer bugs.
- **Robust**: Get production-ready code. With automatic interactive documentation.
- **Standards-based**: Based on (and fully compatible with) the open standards for APIs: [OpenAPI](https://github.com/OAI/OpenAPI-Specification) (previously known as Swagger) and [JSON Schema](https://json-schema.org/).

> **IMPORTANT:** [[path operation methods]]

# getting started
```bash
pip install fastapi[all]

// or

pip install fastapi
pip isntall uvicorn[standard]

```
This will install FastAPI library and [[uvicorn]] server



The most basic example
```python
from fastapi import FastAPI # for less load

app = FastAPI()

@app.get("/")
async def get_wishes():
	return {"status": "ok" }
```

Run the server

> NOTE: FastAPI automatically adds swagger (just add `/docs` to the request) 
> And ReDoc with `/redoc`

# Schemas

## OpenAPI schema
OpenAPI [[schema]] is the specification on how to define API.
This schema includes API paths, possible parameters, return type, etc.

`/openapi.json`
```json

{
  "openapi": "3.1.0",
  "info": {
    "title": "FastAPI",
    "version": "0.1.0"
  },
  "paths": {
    "/": {
      "get": {
        "summary": "Root",
        "operationId": "root__get",
        "responses": {
          "200": {
            "description": "Successful Response",
            "content": {
              "application/json": {
                "schema": {}
              }
            }
          }
        }
      }
    }
  }
}

```
## Data schema
In context of data, schema is the structure or shape of the data itself, like JSON content.
In this case [[schema]] would be a set of attributes of the JSON



## What is OpenAPI schema for?
This OpenAPI [[schema]] is what powers 2 interactive documentations (swagger and redoc)



---

# path operator declarator

Path operators declared via [[decorator]] in python
It tells the `app`, to bind a function with `route` and `operator`, to then navigate on routing
- `@app.get("/")`
- `@app.post()`
- `@app.put()`
- `@app.delete()`

And the more exotic ones:

- `@app.options()`
- `@app.head()`
- `@app.patch()`
- `@app.trace()`


> *NOTE:*, FastAPI doesn't enforce any specific meaning, it is rather a guideline, on which operator/method is to use. E.g. using GraphQL, all actions are done via `POST` 


> NOTE: Routes are matched in order

# return content

We can return any type that cab be JSON parsed (basic types, plus our ORMs, etc.)


---

*Any parameters that are living in a braces* of the handler function, is the parameters, that *will be passed from the user.*

[[path vs query parameters]]
# path parameters
```python
@app.get("/params/{holder}")
async def test_params(holder):
    return holder + "!"
```
parameters will also be passed to the function

also we can enforce the type, and FastAPI will automatically convert a data
`async def test_params(holder: int):`

## predefined values
We can set a list of predefined values, by creating the enum with strings and inheriting from `str`, so that the API docs:
1. will know that the value must be a string
2. render it correctly 

```python
# MRO
class FixedString(str, Enum):
    horse = "horse"
    pig = "pig"
    
    
@app.get("/fixedStrings/{fixed}")
async def fixed_string(fixed: FixedString):
    if fixed == "horse":
        return { "🤠": "you alright boy "}
    return "🐷"


```


## path parameter containing a paths
```python
@app.get("/fixedStrings/{file_path:path}")
async def fixed_string(file_path: str):
    return {"checkout": file_patj}

```
`:path` from Starlette tells that the parameter can match any path.

# query parameters
`http://127.0.0.1:8000/items/?skip=0&limit=10`
```python
@app.get("/testQuery/{path_parameter}")
async def test_query(path_parameter: str, test: str, number: int | None = None):
	if path_parameter:
		return [test, number]
	return test
```

here we can use predefined or [[default parameters]]

> NOTE: when parameter has default value, it is not required


# Request body
To declare a **JSON request body** for the POST or non GET operations, we use [[Pydantic]]'s [[data validation]] BaseModel (to define Pydantic's module, o validate the data)
```python
from pydantic import BaseModel

class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None


@app.post("/items/")
async def create_item(item: Item):
    return item
```

For example, this model above declares a JSON "`object`" (or Python `dict`) like:

```json
{
    "name": "Foo",
    "description": "An optional description",
    "price": 45.2,
    "tax": 3.5
}
```


# Parameter validation and [[metadata]]
Annotating with [[Metadata annotations]] in python, we can


# Ellipsis
In python it `...` is used in different cases, but in Pydantic and FastAPI to explicitly say, that a value is required


# Form annotation (url-encoded)
When we want to receive urlencoded *parameter* instead of JSON, we use annotating parameter with `Form()`

# Response model
When we want to return a [[Pydantic]] model, to automatically document and validate the fields
```python
@app.get("/", response_model=Item)
```


# Request parsing
Also we can manually parse the request with `Request`

```python
from fastapi import Request

@app.get("/items")
def items_handler(request: Request):
	client_host = request.client.host
```


# Dependencies
In FastAPI handling dependencies is done through `Depends` type, that takes callable inside it, so you don't call it directly, FastAPI calls it by itself

```python

def clear_phone(phone: str):
	return re.sub(r"[/D]", "", phone)

@app.get("/items")
def items_handler(phone: Annotated[str | None, Depends(clear_phone)]):
	return phone # cleared phone
```


# Security
For most of the scenarios, we can handle security (authorization. authentication) using `Depends()`, but if we also want to declare a scope, then we can use `Security`


```python
from typing import Annotated
from fastapi import ==Depends==, FastAPI 
from .db import User
from .security import get_current_active_user 

app = FastAPI() 

@app.get("/users/me/items/") 
async def read_own_items( current_user: Annotated[User, Security(get_current_active_user, scopes=["items"])] ): 
	return [{"item_id": "Foo", "owner": current_user.username}]
```
# swagger
accessed by `/docs`
we can extend paths with documentations, by passing parameters into a [[decorator]]


# Router



# Mounting
Adding external, completely independent application to the specific path, that takes care of everything that coming that path

```python
subapi = FastAPI()

@subapi.get("/sub")
def read_sub():
	return "sub"

app.mount("/v2", subapi)
```

> When handling sub-applications, FastAPI handles communication with mount path to the sub-applications using a mechanism of [[ASGI]] specification called "root_path" [[ASGI root_path]]

For example, we can use all Django features with mounting it