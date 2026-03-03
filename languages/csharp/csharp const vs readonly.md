# C# const vs readonly

| | `const` | `readonly` |
|---|---|---|
| Set at | Compile time | Compile time or constructor |
| Implicit | `static` | Instance (unless `static readonly`) |
| Allowed types | Primitives, string, null | Any type |
| Stored as | Literal value inlined at call site | Field reference |

```csharp
public const double Pi = 3.14159;
public readonly DateTime Created = DateTime.Now;
public static readonly int[] Defaults = { 1, 2, 3 };
```

## Sources
- [const keyword](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/const)
- [readonly keyword](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/readonly)
