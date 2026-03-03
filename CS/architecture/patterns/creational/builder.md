# Builder

A creational design pattern that constructs a complex object step by step, separating construction from representation.

## What it does

A builder exposes methods for setting individual parts of an object, each returning `this` for chaining. A final `Build()` (or equivalent) call assembles and returns the completed object.

The construction process is the same regardless of which representation is produced. Different builders can produce different representations using the same construction steps.

## Code

```csharp
public class PizzaBuilder {
    private readonly Pizza _pizza = new();

    public PizzaBuilder SetDough(string dough) { _pizza.Dough = dough; return this; }
    public PizzaBuilder SetSauce(string sauce) { _pizza.Sauce = sauce; return this; }
    public PizzaBuilder AddTopping(string t) { _pizza.Toppings.Add(t); return this; }
    public Pizza Build() => _pizza;
}

var pizza = new PizzaBuilder()
    .SetDough("thin")
    .SetSauce("tomato")
    .AddTopping("cheese")
    .Build();
```

## Sources

- https://refactoring.guru/design-patterns/builder

## Related

- [[factory-method]]
- [[abstract-factory]]
- [[prototype]]

## Process

- How does the builder pattern separate the construction of an object from its final representation?
- What is the role of a Director class in the builder pattern?
- How does builder differ from a constructor with many optional parameters?
