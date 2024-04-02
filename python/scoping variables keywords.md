- `global`: when we defined this, we indicate that nested variable is not a new, but rather a reference to a global variable
- `nonlocal`: we declare that a variable should refer an outer-scoped variable

```python
# Global variable
global_var = 10

def outer_function():
    outer_var = 20

    def inner_function():
        nonlocal outer_var  # Access outer_var from outer_function
        outer_var += 1
        
        global global_var  # Access global_var from global scope
        global_var += 1
        
        print(f"Inner function: outer_var={outer_var}, global_var={global_var}")

    inner_function()
    print(f"Outer function: outer_var={outer_var}, global_var={global_var}")

outer_function()
print(f"Global: global_var={global_var}")
```