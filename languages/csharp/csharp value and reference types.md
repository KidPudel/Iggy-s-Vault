# C# Value and Reference Types

## Value Types

Stored on stack (or inline in containing type). Copy on assignment.

| Type | .NET Type | Size | Range / Notes |
|------|-----------|------|---------------|
| `bool` | `System.Boolean` | 1 byte | `true` / `false` |
| `byte` | `System.Byte` | 1 byte | 0 to 255 |
| `sbyte` | `System.SByte` | 1 byte | -128 to 127 |
| `char` | `System.Char` | 2 bytes | Unicode character |
| `short` | `System.Int16` | 2 bytes | -32,768 to 32,767 |
| `ushort` | `System.UInt16` | 2 bytes | 0 to 65,535 |
| `int` | `System.Int32` | 4 bytes | -2^31 to 2^31-1 |
| `uint` | `System.UInt32` | 4 bytes | 0 to 2^32-1 |
| `long` | `System.Int64` | 8 bytes | -2^63 to 2^63-1 |
| `ulong` | `System.UInt64` | 8 bytes | 0 to 2^64-1 |
| `float` | `System.Single` | 4 bytes | ~6-9 digits precision, suffix `f` |
| `double` | `System.Double` | 8 bytes | ~15-17 digits precision |
| `decimal` | `System.Decimal` | 16 bytes | ~28-29 digits precision, suffix `m` |
| `nint` | `System.IntPtr` | platform | Native-sized integer |
| `nuint` | `System.UIntPtr` | platform | Native-sized unsigned integer |

Also value types: `struct`, `enum`, `ValueTuple`, `Nullable<T>`.

## Reference Types

Stored on heap. Variables hold a reference. `null` by default.

`string`, `object`, `dynamic`, `class`, `interface`, `delegate`, `record class`, arrays, `Func<>`, `Action<>`.

## Default Values

| Type | Default |
|------|---------|
| Numeric types (`int`, `float`, etc.) | `0` |
| `bool` | `false` |
| `char` | `'\0'` |
| `enum` | `0` (first member if it equals 0) |
| `struct` | All fields set to defaults |
| All reference types | `null` |
| `Nullable<T>` | `null` (`HasValue = false`) |

```csharp
int x = default;        // 0
bool b = default;       // false
string s = default;     // null
var d = default(int);   // 0 (explicit form)
```

## Sources
- [Value types](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/value-types)
- [Reference types](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/reference-types)
- [default keyword](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/default)
