```python
from fastapi import FastAPI
app = FastAPI()
```
- This imports specific name explicitly [[Zen of Python]], so we know that is is from module even without using module name accessor


```python
import module

module.x
module._y
```
- imports *entire module namespace as it is* 
- keeps it separated by the module name accessor (dot annotation), this forces explicit action [[Zen of Python]]
- This method of importing *allows accessing module internals when **necessary***, because of the 1st point ("imports *entire module namespace as it is* "), So this is a responsibility to adhere (respect/stick) to the convention and not directly use internal names


```python
from module import *
```
- imports all names from the module, except for internals (`_`)
- makes using names of the module *implicit*, which is not a good practice

```python
from database.long_name_db as db
```

# Relative paths and imports
## Relative (in the same package)
```python
from .forces import Forces
```


## relative from another package
to find other packages not related to current, we can set the path to the parent, so Python will be able to find it