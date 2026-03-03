# C# Lambdas

```csharp
// Expression lambda
Func<int, int> square = x => x * x;
Func<int, int, int> add = (x, y) => x + y;

// Statement lambda
Func<int, int> abs = x =>
{
    if (x < 0) return -x;
    return x;
};

// Discard parameter
Action<int> ignore = _ => Console.WriteLine("fired");

// Static lambda (C# 9+) — cannot capture locals
Func<int, int> f = static x => x * 2;

// Lambda with explicit return type (C# 10+)
var f = int (bool b) => b ? 1 : 0;
```

## Sources
- [Lambda expressions](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/lambda-expressions)
