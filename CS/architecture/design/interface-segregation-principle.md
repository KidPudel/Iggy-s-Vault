# Interface Segregation Principle

A design principle stating that a client should not be forced to depend on methods it does not use.

## What it does

Interfaces should be small and focused on a single set of related behaviors. A class that implements an interface should use all of its methods. When a class implements methods it doesn't need (empty body, throws `NotImplementedException`), the interface is too broad and should be split.

## Code

```csharp
// Violation: Robot forced to implement Eat and Sleep
public interface IWorker { void Work(); void Eat(); void Sleep(); }
public class Robot : IWorker { public void Work() { } public void Eat() => throw new NotImplementedException(); public void Sleep() => throw new NotImplementedException(); }

// Compliant: segregated interfaces
public interface IWorkable { void Work(); }
public interface IFeedable { void Eat(); }
public class Human : IWorkable, IFeedable { public void Work() { } public void Eat() { } }
public class Robot : IWorkable { public void Work() { } }
```

## Sources

- https://en.wikipedia.org/wiki/Interface_segregation_principle

## Related

- [[single-responsibility-principle]]
- [[liskov-substitution-principle]]
- [[dependency-inversion-principle]]
- [[SOLID]]

## Process

- What is the relationship between ISP and SRP at the interface level?
- How does a `NotImplementedException` in an implementing class indicate an ISP violation?
- How does ISP affect the coupling between a client and the classes it depends on?
