to pass the list of the arguments to the function, that takes the number of parameters, we use `*`, to decompose it into an individual different arguments
```python
def test(a, b, c, d, e):
	...
test(*[12, 3, 4], **{d: 2, e: 5})
```
related to [[args and kwargs]]