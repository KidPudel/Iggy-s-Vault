# C# Tuples

```csharp
// Unnamed
(int, string) pair = (1, "one");
pair.Item1;   // 1

// Named
(int id, string name) pair = (1, "one");
pair.id;      // 1

// As return type
public (float min, float max) GetRange() => (0f, 1f);

// Deconstruction
var (min, max) = GetRange();
var (_, max) = GetRange();                 // discard

// In == comparison (C# 7.3+)
(int a, int b) t1 = (1, 2);
(int x, int y) t2 = (1, 2);
bool eq = t1 == t2;                        // true
```

## Sources
- [Tuples](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/value-tuples)
