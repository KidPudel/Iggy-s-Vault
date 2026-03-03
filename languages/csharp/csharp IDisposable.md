# C# IDisposable

```csharp
public class Connection : IDisposable
{
    private bool _disposed;

    public void Dispose()
    {
        Dispose(true);
        GC.SuppressFinalize(this);
    }

    protected virtual void Dispose(bool disposing)
    {
        if (_disposed) return;
        if (disposing)
        {
            // free managed resources
        }
        // free unmanaged resources
        _disposed = true;
    }

    ~Connection() => Dispose(false);
}
```

## using Statements

```csharp
// Block scope
using (var conn = new Connection())
{
    // conn disposed at end of block
}

// Declaration (C# 8+) — disposed at end of enclosing scope
using var conn = new Connection();

// await using for IAsyncDisposable
await using var stream = new AsyncStream();
```

## Sources
- [IDisposable and using](https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/using-objects)
