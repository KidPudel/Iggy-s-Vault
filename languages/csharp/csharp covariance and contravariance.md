# C# Covariance and Contravariance

How inheritance works with generics.

TLDR; `out` means for output and therefore it needs strip down safety. `in` means input therefore implementation needs strip down safety

### Covariance (out) — Can Return Derived Types (Producer)

> Producer widens the gates and catches as wide as we need, as usage will strip down as it needs.

```csharp
// IEnumerable<out T> is the classic producer interface
IEnumerable<string> strings = new List<string> { "Hello" };

// Safe: You can treat a list of strings as a list of objects
// because every item you pull 'out' is guaranteed to be an object.
IEnumerable<object> objects = strings;

foreach (object obj in objects) {
    Console.WriteLine(obj.ToString());
}
```

If implementation states it produces (outputting data `out`) `Dog`s, it's safe to treat it as producing `Animal`s, because every `Dog` is an `Animal`, and returning type of the results of `IEnumerable` will return any of the `Animal`.

implementation set to return `Dog` → the usage catches _any_ `Animal` | END

### Contravariance (in) — Can Accept Base Types (Consumer)

> Consumer tightens the gates as the implementation will strip down to what it can.

```csharp
// Action<in T> is a classic consumer delegate
Action<object> printObject = (obj) => Console.WriteLine(obj.GetHashCode());

// Safe: You can treat an 'object' printer as a 'string' printer.
// The code expecting a string-printer will only ever pass in strings;
// since a string IS an object, the printObject logic handles it perfectly.
Action<string> printString = printObject;

printString("World");
```

If implementation states it consumes (and doing something with it readonly `in`) `Animal`, it is allowed since interface states that using it can handle _up to_ `Dog`s, therefore the implementation itself only will use that part that is == or < than the ceiling.

implementation set to handle `Animal` → the usage passing `Dog` | END

### Practical Example

```csharp
// You have a method that processes animals
void ProcessAnimals(IEnumerable<Animal> animals) { }

// You can pass a list of dogs
List<Dog> dogs = new List<Dog>();
ProcessAnimals(dogs);  // Works because IEnumerable<T> is covariant
```

## Sources
- [Covariance and contravariance](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/covariance-contravariance/)
