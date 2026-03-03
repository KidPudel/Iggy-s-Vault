# Memento

A behavioral design pattern that captures an object's internal state and stores it externally so it can be restored later without violating encapsulation.

## What it does

The originator (the object whose state is saved) creates a memento snapshot containing its internal state. The caretaker stores and manages mementos but never inspects their contents. When a restore is needed, the caretaker passes the memento back to the originator.

Typical uses: undo/redo, snapshots, checkpoints.

## Code

```csharp
public class EditorMemento { public string Content { get; } public EditorMemento(string c) => Content = c; }

public class TextEditor {
    public string Content { get; set; } = "";
    public EditorMemento Save() => new EditorMemento(Content);
    public void Restore(EditorMemento m) => Content = m.Content;
}

var editor = new TextEditor();
var history = new Stack<EditorMemento>();

editor.Content = "Hello";    history.Push(editor.Save());
editor.Content = "Hello World";

editor.Restore(history.Pop());
Console.WriteLine(editor.Content); // Hello
```

## Sources

- https://refactoring.guru/design-patterns/memento

## Related

- [[command]]
- [[state]]

## Process

- How does the memento pattern preserve encapsulation while allowing external state storage?
- What is the caretaker's role and what is it not allowed to do with a memento?
- How does memento combine with command to implement a full undo/redo system?
