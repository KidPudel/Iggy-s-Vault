`with` is used for resource management and exception handing.
It ensures that resources are properly acquired and released, even if the error occurs.

> `with`: carrying it along the scope

`with` is commonly used with the objects that implement [[context manager]] protocol which defines the methods `__enter__()` and `__exit__()`

```python
with expression [as variable]:
	# code block
```

1. `__enter__()` is called and expression returns the object to variable to access in the body
2. if error occurs during the code block execution `__exit__()` with exception details
3. `__exit__()` None if no exception occurred


Also it could be done in a way the `try catch` works, but with `__enter__()` handled in a `with`

> NOTE: It is a good practice to use `with` even with those context managers, which are already opened.
> Just wrap instance in a with, to ensure proper resource management and closing, just in case.
> Example: `with dbconn:`