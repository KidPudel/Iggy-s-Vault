# C# Pattern Matching

## Type Patterns

```csharp
if (obj is string s)             { /* s is string */ }
if (obj is not null)             { /* ... */ }
if (obj is int n and > 0)       { /* ... */ }
```

## Switch Expression (C# 8+)

```csharp
string label = shape switch
{
    Circle { Radius: 0 }       => "point",
    Circle c                   => $"circle r={c.Radius}",
    Rectangle { W: var w, H: var h } when w == h => "square",
    Rectangle r                => $"rect {r.W}x{r.H}",
    null                       => "nothing",
    _                          => "unknown"
};
```

## Property Patterns

```csharp
if (person is { Name: "Alice", Age: >= 18 }) { }

// Nested
if (order is { Customer: { Country: "US" } }) { }

// Extended property pattern (C# 10+)
if (order is { Customer.Country: "US" }) { }
```

## Relational and Logical Patterns (C# 9+)

```csharp
string Grade(int score) => score switch
{
    >= 90 and <= 100 => "A",
    >= 80 and < 90   => "B",
    >= 70 and < 80   => "C",
    < 70 and >= 0    => "F",
    _                => "invalid"
};

// Logical: and, or, not
if (obj is not null and string s) { }
```

## List Patterns (C# 11+)

```csharp
int[] arr = { 1, 2, 3 };

if (arr is [1, 2, 3])                  { }   // exact
if (arr is [1, ..])                    { }   // starts with 1
if (arr is [_, var middle, _])         { }   // capture middle
if (arr is [.., var last])             { }   // capture last
```

## Sources
- [Pattern matching](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/functional/pattern-matching)
