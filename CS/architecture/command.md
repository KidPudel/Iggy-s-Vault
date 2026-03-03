# Command

A behavioral design pattern that encapsulates a request as an object, allowing it to be queued, logged, or undone.

## What it does

A command object holds the receiver (the object that performs the action) and the parameters needed to call it. The invoker holds and executes commands without knowing the receiver. Each command exposes `Execute()` and optionally `Undo()`.

Commands can be stored in a history stack to implement undo/redo.

## Code

```csharp
public interface ICommand { void Execute(); void Undo(); }

public class Light { public void On() => Console.WriteLine("ON"); public void Off() => Console.WriteLine("OFF"); }

public class LightOnCommand : ICommand {
    private readonly Light _light;
    public LightOnCommand(Light l) => _light = l;
    public void Execute() => _light.On();
    public void Undo() => _light.Off();
}

public class RemoteControl {
    private readonly Stack<ICommand> _history = new();
    public void Press(ICommand cmd) { cmd.Execute(); _history.Push(cmd); }
    public void Undo() { if (_history.Count > 0) _history.Pop().Undo(); }
}
```

## Sources

- https://refactoring.guru/design-patterns/command

## Related

- [[memento]]
- [[chain-of-responsibility]]
- [[strategy]]

## Process

- How does encapsulating a request as an object enable undo functionality?
- What is the role of the invoker in the command pattern and what does it not know?
- How does command pattern relate to the concept of a transaction log?
