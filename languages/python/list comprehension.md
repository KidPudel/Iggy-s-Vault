```python
values = [i for i in range(10)]
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```
It works, by taking the value we're spotlighting, in this case `i`, `[look at i, for i in range of 10]`

```python
lists = [[j for j in range(i)] for i in range(10) if i % 2 == 0]
```


```python
[person.speak() for person in people]
```


```python
chars = "AaBbCcDdEeFfGgHhIiJjKkLlMmNnOoPpQqRrSsTtUuVvWwXxYyZz1234567890"
def rand_string(length: int) -> str:
    rnd_chars = [chars[random.randint(0, len(chars) - 1)] for _ in range(length)]
    return ''.join(rnd_chars)
x = [rand_string(i) for i in range(10)]
```