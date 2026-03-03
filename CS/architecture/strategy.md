# Strategy

A behavioral design pattern that defines a family of algorithms, encapsulates each one, and makes them interchangeable at runtime.

## What it does

The context holds a reference to a strategy interface. At runtime, a concrete strategy is injected into the context, and the context delegates the algorithm to it. Swapping a strategy changes the behavior without changing the context.

Strategies are interchangeable because they implement the same interface.

## Code

```csharp
public interface ICompressionStrategy { byte[] Compress(byte[] data); }
public class ZipCompression : ICompressionStrategy { public byte[] Compress(byte[] d) { Console.WriteLine("ZIP"); return d; } }
public class RarCompression : ICompressionStrategy { public byte[] Compress(byte[] d) { Console.WriteLine("RAR"); return d; } }

public class FileCompressor {
    private ICompressionStrategy _strategy;
    public void SetStrategy(ICompressionStrategy s) => _strategy = s;
    public byte[] Compress(byte[] data) => _strategy.Compress(data);
}

var compressor = new FileCompressor();
compressor.SetStrategy(new ZipCompression());
compressor.Compress(data);
```

## Sources

- https://refactoring.guru/design-patterns/strategy

## Related

- [[template-method]]
- [[bridge]]
- [[state]]

## Process

- How does strategy differ from template method in terms of where variation lives?
- What is the relationship between strategy and dependency injection?
- How does strategy compare to bridge structurally, and where do they diverge in intent?
