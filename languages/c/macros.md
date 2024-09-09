Macros are the piece of code indented with [[directives in C]] (`#`) with `define` and then substituted with other *defined* code, by the [[preprocessor]].

```c
#define SQUARE(x) ((x) * (x))
```
When preprocessor encounter `SQUARE(5)`, it will substitute it with `((5) * (5))`

Macros are used for defining constants, code snippets and text substitution (like simple calculation as the example above)


One reason macros is used is performance, they are way of eliminating function call overhead, because they are **always** expanded [[C inline]], as opposite to the `inline` keyword which is a suggestion.

with: `#define` which will be substituted with the constant expression before compilation, and every time the program encounters this macro, it will be replaced with what the macro represents.
```c
#define PI 3.1415
#define circleArea(r) (3.1415*(r)*(r))
```


**BAD**: If just use macros to substitute code, this just will make code unreadable
**GOOD**: But good usage is to automate something, for example like in logging system, where it could be changed

```c
#ifdef PR_DEBUG
#define LOG(x) std::count << x << std::endl;
#else
#define LOG(x)
#endif
```
so this will or print or not depending on mode
but this would be better
```c
#define PR_DEBUG 0
#if PR_DEBUG == 1
#define LOG(x) std::count << x << std::endl;
#else
#define LOG(x)
#endif
```