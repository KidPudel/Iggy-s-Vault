# C# List\<T\>

```csharp
var list = new List<int>();
var list2 = new List<int> { 1, 2, 3 };
var list3 = new List<int>(capacity: 100);
List<int> list4 = [1, 2, 3];         // collection expression (C# 12+)
```

| Member | Signature | Returns |
|--------|-----------|---------|
| `Count` | `int Count` | Element count |
| `Capacity` | `int Capacity` | Current capacity |
| `this[]` | `T this[int index]` | Element at index |
| `Add` | `Add(T item)` | `void` |
| `AddRange` | `AddRange(IEnumerable<T> collection)` | `void` |
| `Insert` | `Insert(int index, T item)` | `void` |
| `InsertRange` | `InsertRange(int index, IEnumerable<T> collection)` | `void` |
| `Remove` | `Remove(T item)` | `bool` |
| `RemoveAt` | `RemoveAt(int index)` | `void` |
| `RemoveAll` | `RemoveAll(Predicate<T> match)` | `int` (count removed) |
| `RemoveRange` | `RemoveRange(int index, int count)` | `void` |
| `Clear` | `Clear()` | `void` |
| `Contains` | `Contains(T item)` | `bool` |
| `IndexOf` | `IndexOf(T item)` | `int` |
| `LastIndexOf` | `LastIndexOf(T item)` | `int` |
| `Find` | `Find(Predicate<T> match)` | `T` |
| `FindAll` | `FindAll(Predicate<T> match)` | `List<T>` |
| `FindIndex` | `FindIndex(Predicate<T> match)` | `int` |
| `Exists` | `Exists(Predicate<T> match)` | `bool` |
| `TrueForAll` | `TrueForAll(Predicate<T> match)` | `bool` |
| `Sort` | `Sort()` | `void` |
| `Sort` | `Sort(Comparison<T> comparison)` | `void` |
| `Reverse` | `Reverse()` | `void` |
| `BinarySearch` | `BinarySearch(T item)` | `int` |
| `ToArray` | `ToArray()` | `T[]` |
| `GetRange` | `GetRange(int index, int count)` | `List<T>` |
| `ForEach` | `ForEach(Action<T> action)` | `void` |
| `AsReadOnly` | `AsReadOnly()` | `ReadOnlyCollection<T>` |

## Sources
- [List\<T\>](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.list-1)
