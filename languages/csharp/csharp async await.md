# C# async/await

```csharp
// Return types: Task, Task<T>, ValueTask, ValueTask<T>, void (event handlers only)
async Task<string> FetchDataAsync(string url)
{
    HttpClient client = new();
    string data = await client.GetStringAsync(url);
    return data;
}

// Calling
string result = await FetchDataAsync("https://example.com");

// Fire and forget (event handler)
async void OnButtonClick(object sender, EventArgs e)
{
    await DoWorkAsync();
}

// Await multiple
await Task.WhenAll(task1, task2, task3);
Task<int> first = await Task.WhenAny(task1, task2);

// ConfigureAwait
await task.ConfigureAwait(false);

// Cancellation
async Task DoWork(CancellationToken ct)
{
    ct.ThrowIfCancellationRequested();
    await Task.Delay(1000, ct);
}

// Task.Run
var result = await Task.Run(() => HeavyComputation());

// Async streams
async IAsyncEnumerable<int> GenerateAsync()
{
    for (int i = 0; i < 10; i++)
    {
        await Task.Delay(100);
        yield return i;
    }
}
await foreach (var item in GenerateAsync()) { }
```

## Sources
- [Async and await](https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/)
