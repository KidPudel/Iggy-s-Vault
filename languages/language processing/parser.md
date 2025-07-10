Parser is the component in a [[syntax analysis]] process that takes the input, and creates the output in a structural representation (typically called parse tree or [[AST]]), and checking for correct syntax along the way.
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