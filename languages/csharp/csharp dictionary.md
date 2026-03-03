# C# Dictionary\<TKey, TValue\>

```csharp
var dict = new Dictionary<string, int>();
var dict2 = new Dictionary<string, int>
{
    { "a", 1 },
    { "b", 2 }
};
var dict3 = new Dictionary<string, int>
{
    ["a"] = 1,
    ["b"] = 2
};
```

| Member | Signature | Returns |
|--------|-----------|---------|
| `Count` | `int Count` | Entry count |
| `Keys` | `KeyCollection Keys` | All keys |
| `Values` | `ValueCollection Values` | All values |
| `this[]` | `TValue this[TKey key]` | Value (throws if missing) |
| `Add` | `Add(TKey key, TValue value)` | `void` (throws if exists) |
| `TryAdd` | `TryAdd(TKey key, TValue value)` | `bool` |
| `Remove` | `Remove(TKey key)` | `bool` |
| `Remove` | `Remove(TKey key, out TValue value)` | `bool` |
| `Clear` | `Clear()` | `void` |
| `ContainsKey` | `ContainsKey(TKey key)` | `bool` |
| `ContainsValue` | `ContainsValue(TValue value)` | `bool` |
| `TryGetValue` | `TryGetValue(TKey key, out TValue value)` | `bool` |
| `GetValueOrDefault` | `GetValueOrDefault(TKey key)` | `TValue` |
| `GetValueOrDefault` | `GetValueOrDefault(TKey key, TValue defaultValue)` | `TValue` |

```csharp
// Iteration
foreach (KeyValuePair<string, int> kvp in dict) { }
foreach (var (key, value) in dict) { }
```

## Sources
- [Dictionary\<TKey, TValue\>](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.dictionary-2)
