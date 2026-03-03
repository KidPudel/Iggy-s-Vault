# Chain of Responsibility

A behavioral design pattern that passes a request along a chain of handlers, where each handler decides to process the request or forward it to the next handler.

## What it does

Each handler holds a reference to the next handler in the chain. When a request arrives, the handler either processes it or calls `next.Handle()`. The sender does not know which handler will process the request.

The chain can be built and modified at runtime by linking handler objects in different orders.

## Code

```csharp
public abstract class Handler {
    protected Handler _next;
    public Handler SetNext(Handler next) { _next = next; return next; }
    public abstract void Handle(string issue, int severity);
}

public class Level1 : Handler {
    public override void Handle(string issue, int severity) {
        if (severity <= 1) Console.WriteLine($"L1: {issue}");
        else _next?.Handle(issue, severity);
    }
}

public class Level2 : Handler {
    public override void Handle(string issue, int severity) {
        if (severity <= 2) Console.WriteLine($"L2: {issue}");
        else _next?.Handle(issue, severity);
    }
}

var chain = new Level1();
chain.SetNext(new Level2());
chain.Handle("Login bug", 2); // L2 handles it
```

## Sources

- https://refactoring.guru/design-patterns/chain-of-responsibility

## Related

- [[mediator]]
- [[command]]
- [[decorator]]

## Process

- What happens to a request when no handler in the chain processes it?
- How does the chain of responsibility differ from an if-else chain in terms of extensibility?
- How does chain of responsibility relate to middleware pipelines in web frameworks?
