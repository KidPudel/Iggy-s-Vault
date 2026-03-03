# State

A behavioral design pattern that allows an object to change its behavior when its internal state changes, appearing to change its class.

## What it does

The context holds a reference to the current state object. State-specific behavior is delegated to the state object. When a transition occurs, the context (or state) replaces the current state with a new one.

Each state class implements the same state interface with the behavior appropriate for that state.

## Code

```csharp
public interface IOrderState { void Next(Order o); void PrintStatus(); }

public class PendingState : IOrderState {
    public void Next(Order o) => o.SetState(new ShippedState());
    public void PrintStatus() => Console.WriteLine("PENDING");
}

public class ShippedState : IOrderState {
    public void Next(Order o) => o.SetState(new DeliveredState());
    public void PrintStatus() => Console.WriteLine("SHIPPED");
}

public class Order {
    private IOrderState _state = new PendingState();
    public void SetState(IOrderState s) => _state = s;
    public void Next() => _state.Next(this);
    public void Print() => _state.PrintStatus();
}
```

## Sources

- https://refactoring.guru/design-patterns/state

## Related

- [[strategy]]
- [[command]]
- [[observer]]

## Process

- How does state differ from strategy when both swap behavior at runtime via an interface?
- Who is responsible for triggering state transitions — the context or the state object?
- How does the state pattern eliminate large `switch` or `if-else` blocks based on state?
