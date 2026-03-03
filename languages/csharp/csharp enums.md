# C# Enums

```csharp
public enum Direction
{
    North,          // 0
    East,           // 1
    South,          // 2
    West            // 3
}

public enum Status : byte
{
    Idle = 0,
    Running = 1,
    Done = 2
}
```

## [Flags] Enum

```csharp
[Flags]
public enum Layers
{
    None    = 0,
    Ground  = 1 << 0,   // 1
    Player  = 1 << 1,   // 2
    Enemy   = 1 << 2,   // 4
    Walls   = 1 << 3,   // 8
    All     = Ground | Player | Enemy | Walls
}

// Usage
var mask = Layers.Player | Layers.Enemy;
bool hasPlayer = mask.HasFlag(Layers.Player);    // true
bool hasPlayer = (mask & Layers.Player) != 0;    // equivalent, no allocation
```

## Sources
- [Enums](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/enum)
- [FlagsAttribute](https://learn.microsoft.com/en-us/dotnet/api/system.flagsattribute)
