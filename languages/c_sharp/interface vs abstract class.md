In C#, both define contracts, but they serve different purposes:

**Interface**

- Pure contract — no implementation (though C# 8+ allows default implementations)
- A class can implement multiple interfaces
- No fields, no constructors
- All members are implicitly public

```csharp
public interface IShape
{
    double Area();
    double Perimeter();
}
```

**Abstract Class**

- Can mix abstract members (no implementation) with concrete members (with implementation)
- A class can inherit only one abstract class
- Can have fields, constructors, access modifiers
- Can maintain state

```csharp
public abstract class Shape
{
    protected string Color;
    
    public abstract double Area();
    
    public void Describe() => Console.WriteLine($"A {Color} shape");
}
```

**When to use which:**

Use an **interface** when you need to define a capability that unrelated classes can share (e.g., `IDisposable`, `IComparable`). Think "can do."

Use an **abstract class** when you have a base concept with shared implementation that derived classes should inherit. Think "is a."

**Practical example:** A `Dog` and `Cat` might inherit from abstract class `Animal` (shared state like `Name`, shared behavior like `Eat()`), but both could implement `ITrainable` — an interface that a `Robot` class could also implement despite not being an animal.