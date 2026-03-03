# Visitor

A behavioral design pattern that lets you define a new operation on elements of an object structure without changing the element classes.

## What it does

Each element class implements `Accept(IVisitor)`, which calls back the visitor with itself (`visitor.Visit(this)`). The visitor implements a `Visit` overload for each element type. New operations are added by creating new visitor implementations without modifying element classes.

Uses double dispatch: the runtime type of both the visitor and the element determine the method called.

## Code

```csharp
public interface IVisitor { void Visit(Circle c); void Visit(Rectangle r); }
public interface IShape { void Accept(IVisitor v); }

public class Circle : IShape { public double Radius; public void Accept(IVisitor v) => v.Visit(this); }
public class Rectangle : IShape { public double W, H; public void Accept(IVisitor v) => v.Visit(this); }

public class AreaCalc : IVisitor {
    public double Total;
    public void Visit(Circle c) => Total += Math.PI * c.Radius * c.Radius;
    public void Visit(Rectangle r) => Total += r.W * r.H;
}

var shapes = new List<IShape> { new Circle { Radius = 5 }, new Rectangle { W = 4, H = 6 } };
var calc = new AreaCalc();
foreach (var s in shapes) s.Accept(calc);
Console.WriteLine(calc.Total);
```

## Sources

- https://refactoring.guru/design-patterns/visitor

## Related

- [[composite]]
- [[iterator]]
- [[strategy]]

## Process

- What is double dispatch and how does the visitor pattern implement it?
- What is the cost of adding a new element type to the visitor pattern?
- How does visitor differ from adding methods directly to each element class?
