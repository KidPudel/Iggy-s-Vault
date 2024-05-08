[Pydantic](https://docs.pydantic.dev/latest/): is the library for deep data validation

if validation fails, pydantic will raise an error with a breakdown of what is wrong.

```python
from datetime import datetime

from pydantic import BaseModel, PositiveInt


class User(BaseModel):
    id: int  
    name: str = 'John Doe'  
    signup_ts: datetime | None  
    tastes: dict[str, PositiveInt]  


external_data = {
    'id': 123,
    'signup_ts': '2019-06-01 12:22',  
    'tastes': {
        'wine': 9,
        b'cheese': 7,  
        'cabbage': '1',  
    },
}

user = User(**external_data)

print(user.model_dump()) # convert to dict
"""
{
    'id': 123,
    'name': 'John Doe',
    'signup_ts': datetime.datetime(2019, 6, 1, 12, 22),
    'tastes': {'wine': 9, 'cheese': 7, 'cabbage': 1},
}
"""

```

# [models](https://docs.pydantic.dev/latest/concepts/models/)
To define the *base* structure of your data *model* -use `BaseModel`
It is a struct, or requirement, which is similar to dataclasses, but in more data validation way
# [fields](https://docs.pydantic.dev/latest/concepts/fields/#default-values)
`Field` function is used to customize and add metadata to the fields of the model

```python
class User(BaseModel):
	id: PositiveInt
	name: str = "unknown"
	calculated_uuid: Field(default_factory=lambda: uuid4().hex)
	
	
	
```
 The `PositiveInt` type is shorthand for `Annotated[int, annotated_types.Gt(0)]`.