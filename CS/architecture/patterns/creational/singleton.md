# Singleton

A creational design pattern that restricts a class to having only one instance and provides a global access point to it.

## What it does

- The constructor is made private, preventing direct instantiation.
- A static method or property controls access to the single instance.
- On first call, it creates the instance; on subsequent calls, it returns the already-created one.
- In multithreaded environments, a lock prevents two threads from creating the instance simultaneously.

## Code (if applicable)

```csharp
public sealed class Logger
{
    private static Logger _instance;
    private static readonly object _lock = new();

    private Logger() { }

    public static Logger Instance
    {
        get
        {
            lock (_lock)
            {
                _instance ??= new Logger();
                return _instance;
            }
        }
    }

    public void Log(string message) => Console.WriteLine($"[LOG] {message}");
}

Logger.Instance.Log("Application started");
```

## Sources

- [https://refactoring.guru/design-patterns/singleton](https://refactoring.guru/design-patterns/singleton)

## Related

- [[factory-method]]
- [[abstract-factory]]
- [[builder]]
- [[design-patterns]]

## Process

- What breaks if two threads both check `_instance == null` before either assigns it?
- How does Singleton behavior change if the class is not `sealed`?
- Where does Singleton interact with dependency injection containers, and what conflict arises?
- What happens to tests that rely on global state set by one Singleton instance?
