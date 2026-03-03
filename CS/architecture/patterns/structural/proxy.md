# Proxy

A structural design pattern that provides a surrogate object controlling access to another object.

## What it does

A proxy implements the same interface as the real object and intercepts calls before forwarding them. The proxy can add behavior such as lazy initialization, access control, logging, or caching without changing the real object.

Common proxy types:
- **Virtual proxy**: defers creation of the real object until first use
- **Protection proxy**: controls access based on permissions
- **Logging proxy**: records calls to the real object

## Code

```csharp
public interface IImage { void Display(); }

public class RealImage : IImage {
    public RealImage(string file) { Console.WriteLine($"Loading {file}"); }
    public void Display() => Console.WriteLine("Displaying");
}

public class ImageProxy : IImage {
    private readonly string _file;
    private RealImage _real;
    public ImageProxy(string file) => _file = file;
    public void Display() {
        _real ??= new RealImage(_file); // lazy load
        _real.Display();
    }
}
```

## Sources

- https://refactoring.guru/design-patterns/proxy

## Related

- [[decorator]]
- [[adapter]]
- [[facade]]

## Process

- What distinguishes a virtual proxy from a protection proxy in terms of purpose?
- How does proxy differ from decorator when both wrap an object behind the same interface?
- What is the proxy pattern's relationship to lazy initialization?
