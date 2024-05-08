Macros are the piece of code indented with [[directives in C]] (`#`) with `define` and then substituted with other *defined* code, by the [[preprocessor]].

```c
#define SQUARE(x) ((x) * (x))
```
When preprocessor encounter `SQUARE(5)`, it will substitute it with `((5) * (5))`

Macros are used for defining constants, code snippets and text substitution (like simple calculation as the example above)

