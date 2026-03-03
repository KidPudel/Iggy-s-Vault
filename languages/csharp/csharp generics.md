# C# Generics

```csharp
public class Pool<T> where T : new()
{
    private readonly List<T> _items = new();
    public T Get() => _items.Count > 0 ? _items[^1] : new T();
}

public interface IRepository<T> where T : class
{
    T GetById(int id);
}

public T Max<T>(T a, T b) where T : IComparable<T>
    => a.CompareTo(b) >= 0 ? a : b;
```

## Constraints

| Constraint | Meaning |
|---|---|
| `where T : struct` | T must be a non-nullable value type |
| `where T : class` | T must be a reference type |
| `where T : class?` | T must be a nullable reference type |
| `where T : notnull` | T must be non-nullable |
| `where T : unmanaged` | T must be an unmanaged type (no references) |
| `where T : new()` | T must have a parameterless constructor (listed last) |
| `where T : BaseClass` | T must derive from BaseClass |
| `where T : IInterface` | T must implement IInterface |
| `where T : U` | T must derive from or be U (type parameter constraint) |
| `where T : default` | Disambiguates overrides with unconstrained T (C# 9+) |

Multiple constraints: `where T : class, IDisposable, new()`

## Sources
- [Generics](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/generics)
