# Open-Closed Principle

A class should be open for extension but closed for modification.

## What it does

- New behavior is added by writing new code (new classes, new implementations), not by editing existing code
- Existing, tested code remains untouched when requirements grow
- Achieved through abstractions: interfaces, abstract classes, delegates, or events that new implementations plug into
- A switch-on-type pattern that grows with every new type is a direct violation
- The "closed" part protects stable code; the "open" part enables growth

## Code (if applicable)

```csharp
// Violation: adding a new discount type requires modifying this class
public class DiscountCalculator
{
    public decimal Calculate(string type, decimal price)
    {
        switch (type)
        {
            case "seasonal": return price * 0.9m;
            case "clearance": return price * 0.5m;
            default: return price;
        }
    }
}

// Compliant: new discount = new class, existing code unchanged
public interface IDiscount { decimal Apply(decimal price); }
public class SeasonalDiscount : IDiscount { public decimal Apply(decimal price) => price * 0.9m; }
public class ClearanceDiscount : IDiscount { public decimal Apply(decimal price) => price * 0.5m; }
public class LoyaltyDiscount : IDiscount { public decimal Apply(decimal price) => price * 0.85m; }

public class DiscountCalculator
{
    public decimal Calculate(IDiscount discount, decimal price) => discount.Apply(price);
}
```

## Sources

- https://en.wikipedia.org/wiki/Open%E2%80%93closed_principle

## Related

- [[single-responsibility-principle]]
- [[liskov-substitution-principle]]
- [[dependency-inversion-principle]]
- [[SOLID]]

## Process

- A codebase uses a large switch statement that is modified every time a new payment method is added. Which principle is being violated, and what structure would fix it?
- OCP says "closed for modification" — does this mean you can never edit a class again once published?
- How does OCP relate to the Liskov Substitution Principle — what does one require of the other?
- When is it premature to apply OCP, and what signals tell you it's time?
