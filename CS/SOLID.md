# SOLID Principles

---

## Single Responsibility Principle

A class should have **one reason to change**. A method should have **one purpose**.

```csharp
// ❌ BAD: Multiple reasons to change
public class Employee
{
    public decimal CalculatePay() { /* payroll logic */ }
    public void SaveToDatabase() { /* persistence logic */ }
    public string GenerateReport() { /* formatting logic */ }
}

// ✅ GOOD: Each class has one job
public class Employee
{
    public string Name { get; set; }
    public decimal HourlyRate { get; set; }
}

public class PayCalculator
{
    public decimal CalculatePay(Employee emp, int hours) => emp.HourlyRate * hours;
}

public class EmployeeRepository
{
    public void Save(Employee emp) { /* database only */ }
}

public class EmployeeReportGenerator
{
    public string Generate(Employee emp) { /* formatting only */ }
}
```

**Ask yourself:** "If X changes, do I need to modify this class?" If you have multiple X's, split it.

---

## Open-Closed Principle

Classes should be **open for extension** but **closed for modification**.

Add new behavior by adding new code, not changing existing code.

```csharp
// ❌ BAD: Must modify class to add new discount type
public class DiscountCalculator
{
    public decimal Calculate(string type, decimal price)
    {
        switch (type)
        {
            case "seasonal": return price * 0.9m;
            case "clearance": return price * 0.5m;
            // Adding new type = modifying this class
            default: return price;
        }
    }
}

// ✅ GOOD: Extend without modifying
public interface IDiscount
{
    decimal Apply(decimal price);
}

public class SeasonalDiscount : IDiscount
{
    public decimal Apply(decimal price) => price * 0.9m;
}

public class ClearanceDiscount : IDiscount
{
    public decimal Apply(decimal price) => price * 0.5m;
}

// New discount = new class, no existing code changes
public class LoyaltyDiscount : IDiscount
{
    public decimal Apply(decimal price) => price * 0.85m;
}

public class DiscountCalculator
{
    public decimal Calculate(IDiscount discount, decimal price) => discount.Apply(price);
}
```

**Extension points:** Abstract classes, interfaces, delegates, events.

---

## Liskov Substitution Principle

Subtypes must be **substitutable** for their base types without breaking the program.

If it looks like a duck but needs batteries, your abstraction is wrong.

```csharp
// ❌ BAD: Square breaks Rectangle's contract
public class Rectangle
{
    public virtual int Width { get; set; }
    public virtual int Height { get; set; }
    public int Area => Width * Height;
}

public class Square : Rectangle
{
    public override int Width
    {
        set { base.Width = base.Height = value; }  // Surprises callers
    }
    public override int Height
    {
        set { base.Width = base.Height = value; }  // Breaks expectations
    }
}

// This fails with Square:
Rectangle rect = new Square();
rect.Width = 5;
rect.Height = 10;
// Expected: Area = 50
// Actual: Area = 100 (both became 10)

// ✅ GOOD: Separate abstractions
public interface IShape
{
    int Area { get; }
}

public class Rectangle : IShape
{
    public int Width { get; set; }
    public int Height { get; set; }
    public int Area => Width * Height;
}

public class Square : IShape
{
    public int Side { get; set; }
    public int Area => Side * Side;
}
```

**Rules:**

- Don't throw unexpected exceptions in subclasses
- Don't strengthen preconditions (require more)
- Don't weaken postconditions (promise less)
- Preserve base class invariants

---

## Interface Segregation Principle

Clients should not be forced to depend on methods they don't use. **Keep interfaces small and focused.**

```csharp
// ❌ BAD: Monolithic interface
public interface IWorker
{
    void Work();
    void Eat();
    void Sleep();
    void AttendMeeting();
    void WriteReport();
}

public class Robot : IWorker
{
    public void Work() { /* ok */ }
    public void Eat() { throw new NotImplementedException(); }      // Robots don't eat
    public void Sleep() { throw new NotImplementedException(); }    // Robots don't sleep
    public void AttendMeeting() { /* ok */ }
    public void WriteReport() { /* ok */ }
}

// ✅ GOOD: Segregated interfaces
public interface IWorkable
{
    void Work();
}

public interface IFeedable
{
    void Eat();
}

public interface ISleepable
{
    void Sleep();
}

public class Human : IWorkable, IFeedable, ISleepable
{
    public void Work() { }
    public void Eat() { }
    public void Sleep() { }
}

public class Robot : IWorkable
{
    public void Work() { }
    // No forced empty implementations
}
```

**Signs you need to split:** `NotImplementedException`, empty methods, "this doesn't apply to me" comments.

---

## Dependency Inversion Principle

High-level modules should not depend on low-level modules. **Both should depend on abstractions.**

```csharp
// ❌ BAD: High-level depends on low-level concrete class
public class SqlDatabase
{
    public void Save(string data) { /* SQL-specific */ }
}

public class UserService  // High-level
{
    private readonly SqlDatabase _db = new SqlDatabase();  // Tight coupling
    
    public void CreateUser(string name)
    {
        _db.Save(name);  // Can't change database without modifying this class
    }
}

// ✅ GOOD: Both depend on abstraction
public interface IDatabase
{
    void Save(string data);
}

public class SqlDatabase : IDatabase
{
    public void Save(string data) { /* SQL */ }
}

public class MongoDatabase : IDatabase
{
    public void Save(string data) { /* Mongo */ }
}

public class UserService
{
    private readonly IDatabase _db;
    
    public UserService(IDatabase db)  // Inject dependency
    {
        _db = db;
    }
    
    public void CreateUser(string name)
    {
        _db.Save(name);  // Works with any IDatabase
    }
}

// Usage - swap implementations freely
var service1 = new UserService(new SqlDatabase());
var service2 = new UserService(new MongoDatabase());
var serviceTest = new UserService(new MockDatabase());  // Easy testing
```

**Key insight:** The abstraction (interface) is owned by the high-level module, not the low-level one. High-level defines what it needs, low-level implements it.

---

## Quick Reference

| Principle                 | One-Liner                  | Code Smell                                   |
| ------------------------- | -------------------------- | -------------------------------------------- |
| **S**ingle Responsibility | One reason to change       | God classes, "and" in class name             |
| **O**pen-Closed           | Extend, don't modify       | Switch on type, frequent edits to same class |
| **L**iskov Substitution   | Subtypes are substitutable | `NotSupportedException`, broken expectations |
| **I**nterface Segregation | Small, focused interfaces  | Empty methods, "doesn't apply" comments      |
| **D**ependency Inversion  | Depend on abstractions     | `new` inside classes, can't unit test        |
