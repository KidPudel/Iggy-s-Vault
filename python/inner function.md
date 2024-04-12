Functions that are generated inside an enclosing function, with which we can implement [[closures]]

They allow
- Organize/structure your code
- allows keeping outer data private (including inner functions) and not make this data global

```python
def calculate(num1, num2, operation):
    def add(a, b):
        return a + b

    def subtract(a, b):
        return a - b

    def multiply(a, b):
        return a * b

    def divide(a, b):
        if b == 0:
            return "Error: Division by zero"
        else:
            return a / b

    operations = {
        '+': add,
        '-': subtract,
        '*': multiply,
        '/': divide
    }

    if operation in operations:
        op_func = operations[operation]
        result = op_func(num1, num2)
        return result
    else:
        return "Invalid operation"

print(calculate(10, 5, '+'))   # Output: 15
print(calculate(10, 5, '-'))   # Output: 5
print(calculate(10, 5, '*'))   # Output: 50
print(calculate(10, 5, '/'))   # Output: 2.0
print(calculate(10, 0, '/'))   # Output: Error: Division by zero
print(calculate(10, 5, '%'))   # Output: Invalid operation
```