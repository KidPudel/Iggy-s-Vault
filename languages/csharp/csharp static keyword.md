# C# static Keyword

```csharp
static class Utility                       // cannot be instantiated, all members static
{
    public static int Count { get; set; }
    public static void Reset() { }
}

class MyClass
{
    public static int InstanceCount;
    static MyClass() { }                   // static constructor, runs once
}
```

## Sources
- [static keyword](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/static)
