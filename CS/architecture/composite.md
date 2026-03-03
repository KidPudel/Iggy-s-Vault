# Composite

A structural design pattern that composes objects into tree structures and lets clients treat individual objects and compositions uniformly.

## What it does

A component interface is implemented by both leaf nodes (single items) and composite nodes (containers of other components). A composite holds a list of children and delegates operations to them. Clients operate on the component interface without distinguishing between leaves and composites.

Typical uses: file systems, UI hierarchies, organization charts.

## Code

```csharp
public abstract class FileSystemItem { public abstract long GetSize(); }

public class File : FileSystemItem {
    public long Size { get; set; }
    public override long GetSize() => Size;
}

public class Folder : FileSystemItem {
    private readonly List<FileSystemItem> _items = new();
    public void Add(FileSystemItem item) => _items.Add(item);
    public override long GetSize() => _items.Sum(i => i.GetSize());
}
```

## Sources

- https://refactoring.guru/design-patterns/composite

## Related

- [[decorator]]
- [[iterator]]
- [[flyweight]]

## Process

- How does the composite pattern allow client code to avoid `is` checks or type casts when traversing a tree?
- What is the trade-off of making the child management methods (`Add`, `Remove`) part of the component interface vs the composite class only?
- How does composite relate to the recursive structure of tree-shaped data?
