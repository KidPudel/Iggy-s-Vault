# Prototype

A creational design pattern that creates new objects by cloning an existing object (the prototype).

## What it does

Each class that acts as a prototype implements a `Clone()` method that returns a copy of itself. The caller receives a new instance without knowing the concrete class of the object being copied.

Cloning avoids re-running expensive initialization. The prototype registry stores pre-configured objects that can be cloned on demand.

## Code

```csharp
public abstract class Shape { public abstract Shape Clone(); }

public class Circle : Shape {
    public int Radius { get; set; }
    public override Shape Clone() => new Circle { Radius = Radius };
}

var original = new Circle { Radius = 5 };
var copy = (Circle)original.Clone();
copy.Radius = 10; // original unaffected
```

## Sources

- https://refactoring.guru/design-patterns/prototype

## Related

- [[factory-method]]
- [[abstract-factory]]
- [[builder]]

## Process

- What is the difference between a shallow clone and a deep clone in the prototype pattern?
- How does the prototype pattern avoid coupling the caller to the concrete class of the object being cloned?
- When would you use a prototype registry and what problem does it solve?
