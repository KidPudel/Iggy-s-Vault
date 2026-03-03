# C# Extension Methods

```csharp
public static class StringExtensions
{
    public static bool IsNullOrEmpty(this string s)
        => string.IsNullOrEmpty(s);

    public static T OrDefault<T>(this T? value, T fallback) where T : struct
        => value ?? fallback;
}

// Usage
"hello".IsNullOrEmpty();
```

Must be in a `static` class, first parameter prefixed with `this`.

## Sources
- [Extension methods](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/extension-methods)
