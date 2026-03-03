# Observer

A behavioral design pattern that defines a one-to-many dependency so that when one object changes state, all its dependents are notified automatically.

## What it does

A subject (publisher) maintains a list of observers (subscribers). When the subject's state changes, it calls a notification method on each observer. Observers register and deregister themselves at runtime.

The subject does not know the concrete types of its observers — only that they implement the observer interface.

## Code

```csharp
public class Stock {
    public event Action<string, decimal> OnPriceChanged;
    private decimal _price;
    public decimal Price {
        get => _price;
        set { if (_price != value) { _price = value; OnPriceChanged?.Invoke("AAPL", _price); } }
    }
}

var stock = new Stock();
stock.OnPriceChanged += (sym, price) => Console.WriteLine($"{sym}: {price}");
stock.Price = 155;
```

## Sources

- https://refactoring.guru/design-patterns/observer

## Related

- [[mediator]]
- [[event-driven architecture]]
- [[command]]

## Process

- How does observer differ from mediator in terms of who knows whom?
- What problem arises when an observer holds a strong reference to the subject and the subject outlives the observer?
- How does the observer pattern relate to the publish-subscribe pattern?
