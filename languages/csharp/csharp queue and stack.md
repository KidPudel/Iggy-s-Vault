# C# Queue\<T\> and Stack\<T\>

## Queue\<T\>

```csharp
var queue = new Queue<int>();
```

| Member | Signature | Returns |
|--------|-----------|---------|
| `Count` | `int Count` | Element count |
| `Enqueue` | `Enqueue(T item)` | `void` |
| `Dequeue` | `Dequeue()` | `T` (throws if empty) |
| `TryDequeue` | `TryDequeue(out T result)` | `bool` |
| `Peek` | `Peek()` | `T` (throws if empty) |
| `TryPeek` | `TryPeek(out T result)` | `bool` |
| `Contains` | `Contains(T item)` | `bool` |
| `Clear` | `Clear()` | `void` |
| `ToArray` | `ToArray()` | `T[]` |

## Stack\<T\>

```csharp
var stack = new Stack<int>();
```

| Member | Signature | Returns |
|--------|-----------|---------|
| `Count` | `int Count` | Element count |
| `Push` | `Push(T item)` | `void` |
| `Pop` | `Pop()` | `T` (throws if empty) |
| `TryPop` | `TryPop(out T result)` | `bool` |
| `Peek` | `Peek()` | `T` (throws if empty) |
| `TryPeek` | `TryPeek(out T result)` | `bool` |
| `Contains` | `Contains(T item)` | `bool` |
| `Clear` | `Clear()` | `void` |
| `ToArray` | `ToArray()` | `T[]` |

## Sources
- [Queue\<T\>](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.queue-1)
- [Stack\<T\>](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.stack-1)
