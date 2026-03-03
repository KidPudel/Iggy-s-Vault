`*args` allows you to pass any number of *positional variables*
`**kwargs` allows you to pass any number of *keyworded variables*

> NOTE: We can use it both ways.
> 1. when declaring a function
> 2. when passing an arguments

```python
def foo(x, y, z):
	...
l = [1, 2, 3]
foo(*l)

d = {"x": 1, "y": 2, "z": 3}

foo(**d)
```