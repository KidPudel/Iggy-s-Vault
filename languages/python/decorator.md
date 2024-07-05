`@something`
You put it on top of a function, like pretty *decorative* had, that's why it

**Decorator takes decorated function *in* and does something with it**. 
Semantically, decorator can add additional features, like a wrapper in a candy to add some more depth to it, or it can treat decorated functions, like handers to do some work for a bigger picture job

For example tells [[fastAPI]] that function corresponds to the specific *path* and method/*operator*

So it is the *path operator decorator*

```python
def comment_on_function(decorated_function):
    def wrapper(*args, **kwargs):
        nonlocal decorated_function
        result = decorated_function(*args, **kwargs)
        print("your function " + decorated_function.__name__ + " returns: " + str(result))

        return result
    # return a nested function to return/ trigger with functionality 
    return wrapper

@comment_on_function
def add_two(x: int, y: int) -> int:
    return x + y

if __name__ == "__main__":
    print(add_two(5, 2))

# your function add_two returns: 7
# 7
```

So it wraps our function with some additional logic, but we want to return our function, but altered, thats what and why is wrapper nested function
Also decorators can take additional arguments to flex the behaviour of the decorator 
And also we can have decorator inside a decorator
So the returning trace will be wrapper1(wrapper2withadditions(decorated_function))