# C# Access Modifiers

| Modifier | Access |
|---|---|
| `public` | No restrictions |
| `private` | Containing class/struct only |
| `protected` | Containing class + derived classes |
| `internal` | Same assembly only |
| `protected internal` | Same assembly OR derived classes (union) |
| `private protected` | Containing class + derived classes in same assembly (intersection) |
| `file` | Same source file only (C# 11+) |

Default access:
- Class/struct members: `private`
- Top-level types: `internal`
- Interface members: `public`
- Enum members: `public`

## Sources
- [Access modifiers](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/access-modifiers)
