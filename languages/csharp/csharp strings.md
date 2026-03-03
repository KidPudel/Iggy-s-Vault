# C# Strings

## String Literals

```csharp
string s1 = "hello\nworld";              // regular (escape sequences)
string s2 = @"C:\Users\name\file.txt";   // verbatim (no escapes, "" for quote)
string s3 = @"line 1
line 2";                                  // verbatim multiline

string s4 = $"Hello, {name}!";           // interpolated
string s5 = $@"Path: C:\{folder}\{file}"; // interpolated + verbatim
string s6 = @$"Path: C:\{folder}\{file}"; // same (either order)

// Raw string literals (C# 11+)
string s7 = """
    This is a "raw" string.
    No escaping needed.
    """;
string s8 = $$"""{ "name": "{{name}}" }""";  // $$ means {{ }} for interpolation
```

## Interpolation Formatting

```csharp
$"{value:C}"       // currency: $1,234.56
$"{value:N2}"      // number with 2 decimals: 1,234.56
$"{value:F3}"      // fixed-point 3 decimals: 1234.560
$"{value:P1}"      // percent: 85.0%
$"{value:D5}"      // pad integer to 5 digits: 00042
$"{value:X}"       // hexadecimal: FF
$"{value,10}"      // right-align in 10-char field
$"{value,-10}"     // left-align in 10-char field
$"{value,10:N2}"   // align + format
```

## Common String Members

| Member | Signature | Returns |
|--------|-----------|---------|
| `Length` | `int Length` | Character count |
| `Substring` | `Substring(int start)` | `string` |
| `Substring` | `Substring(int start, int length)` | `string` |
| `IndexOf` | `IndexOf(string value)` | `int` (-1 if not found) |
| `LastIndexOf` | `LastIndexOf(string value)` | `int` |
| `Contains` | `Contains(string value)` | `bool` |
| `StartsWith` | `StartsWith(string value)` | `bool` |
| `EndsWith` | `EndsWith(string value)` | `bool` |
| `Replace` | `Replace(string old, string new)` | `string` |
| `Remove` | `Remove(int start, int count)` | `string` |
| `Insert` | `Insert(int index, string value)` | `string` |
| `ToUpper` | `ToUpper()` | `string` |
| `ToLower` | `ToLower()` | `string` |
| `Trim` | `Trim()` | `string` |
| `TrimStart` | `TrimStart()` | `string` |
| `TrimEnd` | `TrimEnd()` | `string` |
| `PadLeft` | `PadLeft(int totalWidth, char c)` | `string` |
| `PadRight` | `PadRight(int totalWidth, char c)` | `string` |
| `Split` | `Split(char separator)` | `string[]` |
| `ToCharArray` | `ToCharArray()` | `char[]` |

| Static Method | Signature | Returns |
|---------------|-----------|---------|
| `string.IsNullOrEmpty` | `IsNullOrEmpty(string? value)` | `bool` |
| `string.IsNullOrWhiteSpace` | `IsNullOrWhiteSpace(string? value)` | `bool` |
| `string.Join` | `Join(string separator, IEnumerable values)` | `string` |
| `string.Concat` | `Concat(params string[] values)` | `string` |
| `string.Format` | `Format(string format, params object[] args)` | `string` |
| `string.Compare` | `Compare(string a, string b)` | `int` |

## Range / Index Syntax (C# 8+)

```csharp
string s = "Hello, World!";
s[0]        // 'H'
s[^1]       // '!'         (from end)
s[0..5]     // "Hello"     (range)
s[7..]      // "World!"
s[..5]      // "Hello"
s[^6..^1]   // "World"
```

## StringBuilder

```csharp
var sb = new StringBuilder();
sb.Append("hello");
sb.AppendLine(" world");
sb.Insert(0, "prefix ");
sb.Replace("hello", "hi");
sb.Remove(0, 7);
sb.Clear();
string result = sb.ToString();
```

## Sources
- [Strings](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/strings/)
- [String interpolation](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/tokens/interpolated)
- [Raw string literals](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/reference-types#string-literals)
