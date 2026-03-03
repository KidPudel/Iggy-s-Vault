# C# Operator Overloading

```csharp
public readonly struct Vec2
{
    public float X { get; init; }
    public float Y { get; init; }

    public static Vec2 operator +(Vec2 a, Vec2 b)
        => new() { X = a.X + b.X, Y = a.Y + b.Y };

    public static Vec2 operator -(Vec2 a, Vec2 b)
        => new() { X = a.X - b.X, Y = a.Y - b.Y };

    public static Vec2 operator *(Vec2 v, float s)
        => new() { X = v.X * s, Y = v.Y * s };

    public static bool operator ==(Vec2 a, Vec2 b)
        => a.X == b.X && a.Y == b.Y;

    public static bool operator !=(Vec2 a, Vec2 b)
        => !(a == b);
}
```

Overloadable operators: `+`, `-`, `*`, `/`, `%`, `&`, `|`, `^`, `<<`, `>>`, `==`, `!=`, `<`, `>`, `<=`, `>=`, `!`, `~`, `++`, `--`, `true`, `false`

Pair requirements: `==`/`!=`, `<`/`>`, `<=`/`>=`

## Implicit / Explicit Conversions

```csharp
public readonly struct Meters
{
    public double Value { get; init; }

    public static implicit operator double(Meters m) => m.Value;
    public static implicit operator Meters(double d) => new() { Value = d };
    public static explicit operator int(Meters m) => (int)m.Value;
}

Meters m = 5.0;         // implicit
double d = m;            // implicit
int i = (int)m;          // explicit required
```

## Sources
- [Operator overloading](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/operator-overloading)
- [User-defined conversion operators](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/user-defined-conversion-operators)
