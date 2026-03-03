# Abstract Factory

A creational design pattern that provides an interface for creating families of related objects without specifying their concrete classes.

## What it does

An abstract factory defines methods for creating each product type in a family (e.g., button, checkbox). Concrete factory implementations produce the product family for a specific variant (e.g., Windows, Mac). Client code uses the factory interface without knowing which concrete products it receives.

Adding a new product family means adding a new concrete factory without touching existing client code.

## Code

```csharp
public interface IUIFactory { IButton CreateButton(); ICheckbox CreateCheckbox(); }

public class WindowsFactory : IUIFactory {
    public IButton CreateButton() => new WindowsButton();
    public ICheckbox CreateCheckbox() => new WindowsCheckbox();
}

public class MacFactory : IUIFactory {
    public IButton CreateButton() => new MacButton();
    public ICheckbox CreateCheckbox() => new MacCheckbox();
}

// Client works with IUIFactory — no knowledge of concrete types
IUIFactory factory = new MacFactory();
factory.CreateButton().Render();
```

## Sources

- https://refactoring.guru/design-patterns/abstract-factory

## Related

- [[factory-method]]
- [[builder]]
- [[singleton]]

## Process

- How does abstract factory differ from factory method in terms of scope and granularity?
- What constraint does adding a new product type (e.g., a new widget) impose on all existing concrete factories?
- How does abstract factory enforce consistency within a product family?
