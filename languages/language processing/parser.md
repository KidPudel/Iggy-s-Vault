Parser (also known as *syntactic analysis*) is the component in a [[syntax analysis]] process that takes the input (or [[token]]s), and creates the output in a structural representation (typically called parse tree or [[AST]]), and checking for correct syntax along the way.
Parser turns the input into a data structure that represents the input. That's all.
It is the same for all parsers, whether it is js `JSON.parse` or a parser in a programming language.

```js
> var input = '{"name": "Thorsten", "age": 28}';
> var output = JSON.parse(input);
> output
{ name: 'Thorsten', age: 28 }
> output.name
'Thorsten'
> output.age
28
```

> NOTE: there are parser generators that exists, that take as an input [[CFG]], and produce parser as the output. This output is code that can be in itself fed in a parser with a source code as an input to produce syntax tree like [[AST]]

Generally there are 2 strategies for writing a parser:
- top-down parsing: starts with constructing root node of the [[AST]] and then descends.
	- Most popular approach is recursive descent parsing. it’s a “top down operator precedence” parser, sometimes called “Pratt parser”, after its inventor Vaughan Pratt.
- bottom-up: the other way around.

