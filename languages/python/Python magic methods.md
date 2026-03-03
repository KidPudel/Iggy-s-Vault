Special predefined methods with 2 underscores in front and end (`__`) that are used for overloading operators and implementing special behavior for the object.

# list of dunder methods
## Object creating and initialization
- `__init__(self, ...)`: used for initialization

## Representation and string conversion
- `__str__(self)`: used for custom string representation form of the object that is used by `str()` and `print()`
- `__repr__(self)`: creates the official string representation of an object
## Attribute access
- `__getattr__(self, name)`: calling when accessing an attribute ***and doesn't exist***
- `__getattribute__(self, name)`: is called before looking for the actual attribute 
- `__setattr__(self, name, value)`: Called when an attribute is assigned a value.
- `__delattr__(self, name)`: Called when an attribute is deleted.

## Operator overloading
- `__add__(self, other)`: defines the rules for `+` operator on the object
- `__sub__(self, other)`: Defines the behavior for the `-` operator.
- `__mul__(self, other)`: Defines the behavior for the `*` operator.
- `__truediv__(self, other)`: Defines the behavior for the `/` operator.
- `__lt__(self, other)`: Defines the behavior for the `<` operator.
- `__le__(self, other)`: Defines the behavior for the `<=` operator.
- `__eq__(self, other)`: Defines the behavior for the `==` operator
- `__ne__(self, other)`: Defines the behavior for the `!=` operator.

## Sequence and iterating
- `__len__(self)`: determines the length of the object, that is used by `len()`
- `__iter__(self)`: returns the iterator for the object
- `__setitem__(self, key, value)`: Defines the behavior for assigning values to indexed elements.
- `__delitem__(self, key)`: Defines the behavior for deleting indexed elements.
- `__contains__(self, item)`: Defines the behavior for the `in` operator.

## Callable object
- `__call(self)`: Defines the behavior on calling the object as a function


## [[context manager]]
- `__enter__(self)`: Defines a behavior when entering `with` statement context
- `__exit__(self)` Defines a behavior when exiting `with` statement context

## Descriptors ([[descriptor protocol]]) 
- `__get__(self, instance, owner)`: Defines the behavior for accessing an attribute. (classes that managed by descriptor protocol)
- `__set__(self, instance, value)`: Defines the behavior for setting an attribute.
- `__delete__(self, instance)`: Defines the behavior for deleting an attribute.
