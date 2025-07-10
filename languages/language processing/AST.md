Abstract Syntax Tree what is called the data structure for internal representation of the source code.

The "Abstract" is based on the fact that certain details visible in the source code are *omitted in the AST*.
Semicolons, newlines, whitespace, comments, braces, bracket and parentheses – depending on the language and the [[parser]], these details may not be represented in the AST, but instead *guide the parse when constructing the data structure*.