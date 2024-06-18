Shallow copy creates a copy of ab object, but references the elements of that object. Meaning of one of the elements is a reference, then this reference will be copied
```python
b = copy.copy(a)
```

Deep copy - recursively creates a a copy of the object and all elements of it
```python
b = copy.deepcopy(a)
```