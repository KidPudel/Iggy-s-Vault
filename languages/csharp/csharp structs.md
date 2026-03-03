# C# Structs

```csharp
public struct Vector2
{
    public float X;
    public float Y;

    public Vector2(float x, float y) { X = x; Y = y; }
}

public readonly struct Point
{
    public float X { get; init; }
    public float Y { get; init; }
}

public ref struct SpanWrapper
{
    public Span<int> Data;
}
```

- Value type, typically stack-allocated
- No inheritance (implicitly sealed), can implement interfaces
- Default value: all fields zeroed
- Equality: value-wise comparison for `readonly record struct`, otherwise reference by default
- `readonly struct` — all members immutable
- `ref struct` — stack-only, cannot be boxed or used in async/iterators

## Sources
- [Structs](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/struct)
