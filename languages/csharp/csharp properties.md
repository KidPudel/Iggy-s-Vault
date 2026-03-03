# C# Properties

## Auto-Property

```csharp
public string Name { get; set; }
public string Name { get; private set; }
public string Name { get; init; }          // init-only (C# 9+)
public string Name { get; } = "default";   // getter-only with initializer
```

## Full Property (backing field)

```csharp
private string _name;
public string Name
{
    get { return _name; }
    set { _name = value; }
}
```

## Expression-Bodied Property

```csharp
public string FullName => $"{First} {Last}";          // get-only
public string Name
{
    get => _name;
    set => _name = value ?? throw new ArgumentNullException();
}
```

## Required Properties (C# 11+)

```csharp
public required string Name { get; set; }
```

## Sources
- [Properties](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/properties)
