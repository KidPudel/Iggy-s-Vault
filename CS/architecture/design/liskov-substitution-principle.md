# Liskov Substitution Principle

A design principle stating that a subtype must be usable in place of its base type without changing the correctness of the program.

## What it does

Any code that works with a base class or interface must continue to work correctly when given a subclass instance. A subclass must not narrow preconditions (require more than the base), weaken postconditions (deliver less), or throw exceptions not declared by the base.

Violation: a subclass that throws `NotImplementedException` for inherited methods, or that overrides behavior in a way that surprises callers expecting base-class contracts.

## Code

```csharp
// Violation: Square breaks Rectangle's width/height contract
public class Rectangle { public virtual int Width { get; set; } public virtual int Height { get; set; } }
public class Square : Rectangle {
    public override int Width { set { base.Width = base.Height = value; } }
}

// Compliant: separate types, shared interface
public interface IShape { int Area { get; } }
public class Rectangle : IShape { public int Width; public int Height; public int Area => Width * Height; }
public class Square : IShape { public int Side; public int Area => Side * Side; }
```

## Sources

- https://en.wikipedia.org/wiki/Liskov_substitution_principle

## Related

- [[single-responsibility-principle]]
- [[open-closed-principle]]
- [[interface-segregation-principle]]
- [[dependency-inversion-principle]]
- [[SOLID]]

## Process

- What does it mean for a subclass to "strengthen a precondition" and why does that violate LSP?
- How does the Square/Rectangle example demonstrate an LSP violation?
- What is the relationship between LSP and the design of interface contracts?
