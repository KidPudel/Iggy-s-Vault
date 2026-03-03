# Mediator

A behavioral design pattern that defines a central object (mediator) that encapsulates how a set of objects communicate, reducing direct dependencies between them.

## What it does

Instead of objects referencing each other directly, they hold a reference to the mediator and communicate through it. The mediator routes messages and coordinates interactions. Objects are decoupled from one another but coupled to the mediator.

Typical uses: chat rooms, UI component coordination, air traffic control.

## Code

```csharp
public interface IMediator { void Send(string msg, User from); }

public class ChatRoom : IMediator {
    private readonly List<User> _users = new();
    public void Register(User u) => _users.Add(u);
    public void Send(string msg, User from) {
        foreach (var u in _users.Where(u => u != from)) u.Receive(msg);
    }
}

public class User {
    private readonly IMediator _mediator;
    public User(IMediator m) => _mediator = m;
    public void Send(string msg) => _mediator.Send(msg, this);
    public void Receive(string msg) => Console.WriteLine(msg);
}
```

## Sources

- https://refactoring.guru/design-patterns/mediator

## Related

- [[observer]]
- [[facade]]
- [[command]]

## Process

- How does mediator reduce coupling between components compared to direct object references?
- What is the trade-off introduced by centralizing all communication in the mediator?
- How does mediator differ from observer in terms of communication direction?
