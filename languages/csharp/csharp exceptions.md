# C# Exceptions

```csharp
try
{
    riskyOperation();
}
catch (FileNotFoundException ex)
{
    // specific exception
}
catch (IOException ex) when (ex.HResult == -2147024864)
{
    // exception filter
}
catch (Exception ex)
{
    throw;            // rethrow preserving stack trace
    // throw ex;      // resets stack trace (avoid)
    // throw new CustomException("msg", ex);  // wrap
}
finally
{
    // always runs
}
```

## Common Exception Types

| Exception | Namespace |
|-----------|-----------|
| `ArgumentException` | `System` |
| `ArgumentNullException` | `System` |
| `ArgumentOutOfRangeException` | `System` |
| `InvalidOperationException` | `System` |
| `NotSupportedException` | `System` |
| `NotImplementedException` | `System` |
| `NullReferenceException` | `System` |
| `IndexOutOfRangeException` | `System` |
| `KeyNotFoundException` | `System.Collections.Generic` |
| `FormatException` | `System` |
| `OverflowException` | `System` |
| `IOException` | `System.IO` |
| `FileNotFoundException` | `System.IO` |
| `TimeoutException` | `System` |
| `TaskCanceledException` | `System.Threading.Tasks` |
| `OperationCanceledException` | `System` |
| `ObjectDisposedException` | `System` |

## Sources
- [Exception handling](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/exceptions/)
