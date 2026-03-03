# Parser

The component in a language processing pipeline that takes tokens as input and produces a structural representation (parse tree or AST) as output, while checking for syntactic correctness.

## What it does

A parser performs syntactic analysis: it consumes a stream of tokens (output of a lexer) and builds a data structure representing the program's structure. The output is typically an Abstract Syntax Tree (AST).

Two main parsing strategies:
- **Top-down**: begins at the root node and descends. Recursive descent / Pratt parsing are the most common forms.
- **Bottom-up**: builds the tree from leaves upward.

Parser generators (e.g., ANTLR, yacc) accept a Context-Free Grammar (CFG) as input and output parser code automatically.

## Code

```js
// Every parser does this — input string → structured data
var input = '{"name": "Thorsten", "age": 28}';
var output = JSON.parse(input);
output.name // 'Thorsten'
```

## Sources

- https://en.wikipedia.org/wiki/Parsing
- https://craftinginterpreters.com/

## Related

- [[AST]]
- [[token]]
- [[lexer]]
- [[grapheme-cluster]]
- [[CFG]]

## Process

- What is the difference between syntactic analysis (parsing) and semantic analysis?
- How does a Pratt parser handle operator precedence compared to a grammar-rule-based parser?
- What is the relationship between a Context-Free Grammar and the parser it generates?
