# Decorator

A structural design pattern that attaches additional responsibilities to an object dynamically by wrapping it in decorator objects that share the same interface.

## What it does

A decorator implements the same interface as the wrapped object and forwards calls to it, adding behavior before or after. Decorators can be stacked: each wraps the previous one, composing behaviors without subclassing.

The wrapped object is unaware it is being decorated.

## Code

```csharp
public interface ICoffee { string Describe(); decimal Cost(); }
public class Coffee : ICoffee { public string Describe() => "Coffee"; public decimal Cost() => 2.00m; }

public abstract class CoffeeDecorator : ICoffee {
    protected readonly ICoffee _inner;
    protected CoffeeDecorator(ICoffee c) => _inner = c;
    public virtual string Describe() => _inner.Describe();
    public virtual decimal Cost() => _inner.Cost();
}

public class MilkDecorator : CoffeeDecorator {
    public MilkDecorator(ICoffee c) : base(c) { }
    public override string Describe() => _inner.Describe() + ", Milk";
    public override decimal Cost() => _inner.Cost() + 0.50m;
}

ICoffee c = new MilkDecorator(new Coffee());
Console.WriteLine(c.Cost()); // 2.50
```

## Sources

- https://refactoring.guru/design-patterns/decorator

## Related

- [[CS/architecture/proxy]]
- [[composite]]
- [[bridge]]

## Process

- How does decorator differ from subclassing for extending an object's behavior?
- What is the effect on the call stack when multiple decorators are stacked?
- How does decorator compare to proxy when both wrap an object behind the same interface?
