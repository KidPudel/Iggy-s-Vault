# C# Records

```csharp
// Record class (reference type, C# 9+)
public record Player(string Name, int Score);

// Record struct (value type, C# 10+)
public record struct Tile(int X, int Y);
public readonly record struct Tile(int X, int Y);

// Non-positional record
public record Config
{
    public string Mode { get; init; }
    public int Level { get; init; }
}
```

Records provide: value-based equality, `ToString()` override, `with` expression, deconstruction (positional records).

```csharp
var p2 = player with { Score = 100 };     // non-destructive mutation
var (name, score) = player;                // deconstruct
```

## Sources
- [Records](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record)
