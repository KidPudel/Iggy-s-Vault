# C# Inheritance Keywords

## virtual / override

```csharp
class Base
{
    public virtual void Speak() { }
    public virtual string Name => "Base";
}

class Derived : Base
{
    public override void Speak() { }
    public override string Name => "Derived";
}
```

## abstract

```csharp
abstract class Shape
{
    public abstract double Area { get; }
    public abstract void Draw();
}

class Circle : Shape
{
    public override double Area => Math.PI * Radius * Radius;
    public override void Draw() { }
}
```

## sealed

```csharp
sealed class Final { }                     // cannot be inherited

class Derived : Base
{
    public sealed override void Speak() { } // cannot be further overridden
}
```

## new (member hiding)

```csharp
class Derived : Base
{
    public new void Speak() { }            // hides Base.Speak (no polymorphism)
}
```

## Sources
- [virtual](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/virtual)
- [override](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/override)
- [abstract](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/abstract)
- [sealed](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/sealed)
- [new modifier](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/new-modifier)
