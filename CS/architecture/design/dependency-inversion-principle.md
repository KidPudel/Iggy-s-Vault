# Dependency Inversion Principle

A design principle stating that high-level modules should not depend on low-level modules; both should depend on abstractions.

## What it does

High-level modules (business logic) define abstractions (interfaces) for what they need. Low-level modules (concrete implementations) implement those abstractions. Neither layer depends directly on the other's concrete type.

This is the principle that motivates dependency injection: dependencies are passed in as abstractions rather than instantiated directly with `new`.

## Code

```csharp
// Violation: UserService coupled to SqlDatabase
public class UserService {
    private readonly SqlDatabase _db = new SqlDatabase(); // tight coupling
}

// Compliant: both depend on IDatabase abstraction
public interface IDatabase { void Save(string data); }
public class SqlDatabase : IDatabase { public void Save(string data) { } }
public class UserService {
    private readonly IDatabase _db;
    public UserService(IDatabase db) { _db = db; } // injected
}
```

## Sources

- https://en.wikipedia.org/wiki/Dependency_inversion_principle

## Related

- [[single-responsibility-principle]]
- [[interface-segregation-principle]]
- [[open-closed-principle]]
- [[SOLID]]
- [[dependency injection]]

## Process

- What is the difference between "dependency inversion" and "dependency injection"?
- Why does the abstraction (interface) belong to the high-level module rather than the low-level module?
- How does DIP affect testability when compared to directly instantiating concrete classes?
