# C# Interfaces

```csharp
public interface IDamageable
{
    int HP { get; set; }
    void TakeDamage(int amount);

    // Default implementation (C# 8+)
    void Kill() => TakeDamage(HP);

    // Static abstract member (C# 11+)
    static abstract IDamageable Create();
}
```

```csharp
// Explicit interface implementation
public class Wall : IDamageable, IRepairable
{
    int IDamageable.HP { get; set; }
    void IDamageable.TakeDamage(int amount) { }
}
```

## Sources
- [Interfaces](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/interfaces)
