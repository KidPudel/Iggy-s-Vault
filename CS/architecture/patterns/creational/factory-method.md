# Factory Method

A creational design pattern that defines an interface for creating an object but lets subclasses decide which class to instantiate.

## What it does

- A base class declares an abstract "factory method" that returns an object of a common interface or abstract type.
- Subclasses override the factory method to return a specific concrete type.
- The base class can call the factory method without knowing which concrete type will be returned.
- The caller works with the product through its interface, not the concrete class.

## Code (if applicable)

```csharp
public abstract class Document { public abstract void Open(); }

public class PdfDocument : Document
{
    public override void Open() => Console.WriteLine("Opening PDF...");
}

public abstract class DocumentCreator
{
    public abstract Document CreateDocument(); // factory method

    public void OpenDocument()
    {
        var doc = CreateDocument();
        doc.Open();
    }
}

public class PdfCreator : DocumentCreator
{
    public override Document CreateDocument() => new PdfDocument();
}

DocumentCreator creator = new PdfCreator();
creator.OpenDocument();
```

## Sources

- [https://refactoring.guru/design-patterns/factory-method](https://refactoring.guru/design-patterns/factory-method)

## Related

- [[abstract-factory]]
- [[singleton]]
- [[builder]]
- [[prototype]]
- [[design-patterns]]

## Process

- What happens if a new document type is added but the creator subclass is not updated?
- How does Factory Method differ from directly calling `new` inside the base class?
- What breaks if the factory method returns a concrete type instead of the interface?
- Where does the Factory Method pattern appear in framework code you have used?
