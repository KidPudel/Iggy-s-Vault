Type annotation or type hints is optional, but is more explicit

```python
def decoding_token(token: str) -> error
```


simple types
- `str`
- `int`
- `float`
- `bool`
- `bytes`
- `list`
- `tuple`
- `set`
- `dict`: map
- union: `status: str | int`
- **Possibly** `None`: 
Pre Python 3.10 using library `types`, we can use `name: Optional[str] = None`, which indicates that it not always `str`, or using `Union[SomeType, None]`
> If you are using a Python version below 3.10, here's a tip from my very **subjective** point of view:
> - 🚨 Avoid using `Optional[SomeType]`
> - Instead ✨ **use `Union[SomeType, None]`** ✨.

Python 3.10+, we can just use 
```python
def print_name(name: str | None = None)
```
This indicates that it takes or string or *None type*, with already default value of `None`, which makes an argument optional