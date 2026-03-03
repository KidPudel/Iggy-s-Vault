# C# Arrays

```csharp
// Declaration and initialization
int[] a = new int[5];
int[] b = new int[] { 1, 2, 3 };
int[] c = { 1, 2, 3 };
int[] d = [1, 2, 3];                // collection expression (C# 12+)

// Multidimensional
int[,] grid = new int[3, 4];
int[,] grid2 = { {1,2}, {3,4} };
grid[0, 1] = 5;

// Jagged
int[][] jagged = new int[3][];
jagged[0] = new int[] { 1, 2 };

// Range/Index
int[] slice = a[1..3];
int last = a[^1];
```

| Member | Signature | Returns |
|--------|-----------|---------|
| `Length` | `int Length` | Total element count |
| `Rank` | `int Rank` | Number of dimensions |
| `GetLength` | `GetLength(int dimension)` | `int` |
| `CopyTo` | `CopyTo(Array dest, int index)` | `void` |
| `Clone` | `Clone()` | `object` (shallow copy) |

| Static Method | Signature | Returns |
|---------------|-----------|---------|
| `Array.Sort` | `Sort<T>(T[] array)` | `void` |
| `Array.Reverse` | `Reverse<T>(T[] array)` | `void` |
| `Array.IndexOf` | `IndexOf<T>(T[] array, T value)` | `int` |
| `Array.Find` | `Find<T>(T[] array, Predicate<T> match)` | `T` |
| `Array.FindAll` | `FindAll<T>(T[] array, Predicate<T> match)` | `T[]` |
| `Array.Exists` | `Exists<T>(T[] array, Predicate<T> match)` | `bool` |
| `Array.Resize` | `Resize<T>(ref T[] array, int newSize)` | `void` |
| `Array.Fill` | `Fill<T>(T[] array, T value)` | `void` |
| `Array.Empty` | `Empty<T>()` | `T[]` |

## Sources
- [Array class](https://learn.microsoft.com/en-us/dotnet/api/system.array)
