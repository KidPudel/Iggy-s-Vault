# Interpreter

A behavioral design pattern that defines a grammar for a language and provides an interpreter to evaluate sentences in that language.

## What it does

Each grammar rule is represented as a class. Terminal expressions (literals) and non-terminal expressions (rules combining sub-expressions) implement a common `Interpret()` method. An abstract syntax tree (AST) is built from these objects and evaluated by calling `Interpret()` on the root.

Typical uses: simple DSLs, expression evaluators, rules engines, configuration parsers.

## Code

```csharp
public interface IExpression { int Interpret(); }

public class Number : IExpression {
    private readonly int _v;
    public Number(int v) => _v = v;
    public int Interpret() => _v;
}

public class Add : IExpression {
    private readonly IExpression _l, _r;
    public Add(IExpression l, IExpression r) { _l = l; _r = r; }
    public int Interpret() => _l.Interpret() + _r.Interpret();
}

// (5 + 10) evaluates to 15
IExpression expr = new Add(new Number(5), new Number(10));
Console.WriteLine(expr.Interpret()); // 15
```

## Sources

- https://refactoring.guru/design-patterns/interpreter

## Related

- [[composite]]
- [[visitor]]
- [[iterator]]

## Process

- How does the interpreter pattern represent grammar rules as a class hierarchy?
- What is the relationship between an abstract syntax tree and the interpreter pattern?
- When does the complexity of the interpreter pattern become impractical relative to using a parser generator?
