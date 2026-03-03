# Iterator

A behavioral design pattern that provides a way to sequentially access elements of a collection without exposing its underlying structure.

## What it does

An iterator object encapsulates the traversal logic and current position. The collection exposes a factory method that returns a new iterator. Clients call `HasNext()` / `Next()` (or language-equivalent) on the iterator without knowing whether the collection is a list, tree, or graph.

In most languages, iterators are built into the language via interfaces like `IEnumerable` / `IEnumerator` (C#) or the iterator protocol (Python, JavaScript).

## Code

```csharp
public class BookCollection : IEnumerable<Book> {
    private readonly List<Book> _books = new();
    public void Add(Book b) => _books.Add(b);
    public IEnumerator<Book> GetEnumerator() => _books.GetEnumerator();
    IEnumerator IEnumerable.GetEnumerator() => GetEnumerator();
}

var library = new BookCollection();
library.Add(new Book { Title = "Design Patterns" });
foreach (var book in library) Console.WriteLine(book.Title);
```

## Sources

- https://refactoring.guru/design-patterns/iterator

## Related

- [[composite]]
- [[visitor]]

## Process

- How does the iterator pattern decouple traversal logic from the collection's internal data structure?
- What is the relationship between the iterator pattern and `yield return` in C# or generators in Python?
- How does an external iterator differ from an internal iterator?
