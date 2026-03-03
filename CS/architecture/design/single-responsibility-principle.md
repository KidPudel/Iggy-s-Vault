# Single Responsibility Principle

A class or module should have exactly one reason to change.

## What it does

- Each class is responsible for one distinct piece of functionality
- If a class handles payroll, it should not also handle database persistence or report generation
- "Reason to change" maps to a single actor or stakeholder whose requirements drive that class
- Splitting concerns means changes in one area (e.g., report format) don't touch unrelated logic (e.g., pay calculation)
- Violation symptoms: class names with "and," god classes, methods that do unrelated things

## Code (if applicable)

```csharp
// Violation: three reasons to change
public class Employee
{
    public decimal CalculatePay() { /* payroll logic */ }
    public void SaveToDatabase() { /* persistence logic */ }
    public string GenerateReport() { /* formatting logic */ }
}

// Each class has one job
public class Employee { public string Name { get; set; } public decimal HourlyRate { get; set; } }
public class PayCalculator { public decimal CalculatePay(Employee emp, int hours) => emp.HourlyRate * hours; }
public class EmployeeRepository { public void Save(Employee emp) { /* database only */ } }
public class EmployeeReportGenerator { public string Generate(Employee emp) { /* formatting only */ } }
```

## Sources

- https://en.wikipedia.org/wiki/Single-responsibility_principle

## Related

- [[open-closed-principle]]
- [[liskov-substitution-principle]]
- [[interface-segregation-principle]]
- [[dependency-inversion-principle]]
- [[SOLID]]

## Process

- If two different teams both need to modify the same class for different reasons, what does that indicate about its design?
- How does SRP interact with cohesion — can a class be highly cohesive but still violate SRP?
- When splitting a class that violates SRP, how do you decide where the boundary goes?
- A class has 10 methods, all related to "user management" — does that automatically mean it follows SRP?
