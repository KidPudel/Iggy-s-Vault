# C# Delegates and Events

## Delegates

```csharp
// Declaration
public delegate int MathOp(int x, int y);

// Instantiation
MathOp add = (x, y) => x + y;
MathOp add = new MathOp(AddMethod);

// Multicast
MathOp combined = add + subtract;
combined -= subtract;

// Invocation
int result = add(3, 4);
int? result = add?.Invoke(3, 4);   // null-safe invoke
```

### Built-in Delegate Types

| Delegate | Signature |
|---|---|
| `Action` | `void ()` |
| `Action<T>` | `void (T)` |
| `Action<T1, T2, ...>` | `void (T1, T2, ...)` up to 16 params |
| `Func<TResult>` | `TResult ()` |
| `Func<T, TResult>` | `TResult (T)` |
| `Func<T1, T2, ..., TResult>` | `TResult (T1, T2, ...)` up to 16 params |
| `Predicate<T>` | `bool (T)` |

## Events

```csharp
// Declaration with EventHandler
public event EventHandler<MyEventArgs> OnDamage;

// Custom event args
public class MyEventArgs : EventArgs
{
    public int Value { get; init; }
}

// Raise
OnDamage?.Invoke(this, new MyEventArgs { Value = 10 });

// Subscribe / unsubscribe
obj.OnDamage += HandleDamage;
obj.OnDamage -= HandleDamage;
void HandleDamage(object sender, MyEventArgs e) { }
```

### Custom Event Accessors

```csharp
private EventHandler _onClick;
public event EventHandler OnClick
{
    add { _onClick += value; }
    remove { _onClick -= value; }
}
```

## Sources
- [Delegates](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/delegates/)
- [Events](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/events/)
