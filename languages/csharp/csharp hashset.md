# C# HashSet\<T\>

```csharp
var set = new HashSet<int> { 1, 2, 3 };
```

| Member | Signature | Returns |
|--------|-----------|---------|
| `Count` | `int Count` | Element count |
| `Add` | `Add(T item)` | `bool` (false if existed) |
| `Remove` | `Remove(T item)` | `bool` |
| `Clear` | `Clear()` | `void` |
| `Contains` | `Contains(T item)` | `bool` |
| `UnionWith` | `UnionWith(IEnumerable<T> other)` | `void` |
| `IntersectWith` | `IntersectWith(IEnumerable<T> other)` | `void` |
| `ExceptWith` | `ExceptWith(IEnumerable<T> other)` | `void` |
| `SymmetricExceptWith` | `SymmetricExceptWith(IEnumerable<T> other)` | `void` |
| `IsSubsetOf` | `IsSubsetOf(IEnumerable<T> other)` | `bool` |
| `IsSupersetOf` | `IsSupersetOf(IEnumerable<T> other)` | `bool` |
| `IsProperSubsetOf` | `IsProperSubsetOf(IEnumerable<T> other)` | `bool` |
| `IsProperSupersetOf` | `IsProperSupersetOf(IEnumerable<T> other)` | `bool` |
| `Overlaps` | `Overlaps(IEnumerable<T> other)` | `bool` |
| `SetEquals` | `SetEquals(IEnumerable<T> other)` | `bool` |
| `TryGetValue` | `TryGetValue(T equalValue, out T actualValue)` | `bool` |

## Sources
- [HashSet\<T\>](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.hashset-1)
