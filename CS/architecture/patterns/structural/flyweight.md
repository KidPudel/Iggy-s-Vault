# Flyweight

A structural design pattern that minimizes memory use by sharing common state across many fine-grained objects.

## What it does

State is divided into intrinsic (shared, stored in the flyweight) and extrinsic (unique per object, passed in at runtime). A flyweight factory ensures that only one flyweight instance exists per unique intrinsic state. Objects store a reference to a flyweight plus their own extrinsic data.

Useful when a large number of objects share most of their state (e.g., characters in a text document, trees in a game forest).

## Code

```csharp
public class TreeType { public string Name; public string Texture; }

public class TreeFactory {
    private static readonly Dictionary<string, TreeType> _cache = new();
    public static TreeType Get(string name, string texture) {
        var key = $"{name}_{texture}";
        if (!_cache.ContainsKey(key)) _cache[key] = new TreeType { Name = name, Texture = texture };
        return _cache[key];
    }
}

// Many Tree objects share a single TreeType instance
public class Tree { public int X, Y; public TreeType Type; }
```

## Sources

- https://refactoring.guru/design-patterns/flyweight

## Related

- [[composite]]
- [[singleton]]
- [[prototype]]

## Process

- What is the difference between intrinsic and extrinsic state in the flyweight pattern?
- How does the flyweight factory enforce sharing of flyweight instances?
- What happens to the flyweight pattern's memory benefit if every object has unique intrinsic state?
