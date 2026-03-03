# C# Classes

```csharp
public class Enemy
{
    public string Name { get; set; }
    public int HP { get; set; }

    public Enemy(string name, int hp)
    {
        Name = name;
        HP = hp;
    }
}
```

- Reference type, allocated on heap
- Supports inheritance (single base class)
- Default value: `null`
- Equality: reference equality by default

## Sources
- [Classes](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/classes)
