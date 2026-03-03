# Template Method

A behavioral design pattern that defines the skeleton of an algorithm in a base class and defers some steps to subclasses.

## What it does

The base class contains the template method — a final method that calls a sequence of steps. Some steps are abstract (subclasses must implement them); others are hooks (optional overrides with a default). The algorithm's structure is fixed in the base class; only the variable steps change.

## Code

```csharp
public abstract class DataProcessor {
    public void Process() { ReadData(); ProcessData(); SaveData(); } // template method

    protected abstract void ReadData();
    protected abstract void ProcessData();
    protected virtual void SaveData() => Console.WriteLine("Saving to default location");
}

public class CsvProcessor : DataProcessor {
    protected override void ReadData() => Console.WriteLine("Reading CSV");
    protected override void ProcessData() => Console.WriteLine("Parsing CSV");
}
```

## Sources

- https://refactoring.guru/design-patterns/template-method

## Related

- [[strategy]]
- [[factory-method]]

## Process

- What is the difference between an abstract step and a hook in a template method?
- How does template method invert control compared to calling subclass methods directly?
- How does template method compare to strategy in terms of where variation is expressed?
