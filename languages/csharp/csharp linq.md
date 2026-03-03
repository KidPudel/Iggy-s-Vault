# C# LINQ

`using System.Linq;`

## Query Syntax vs Method Syntax

```csharp
// Query syntax
var result = from n in numbers
             where n > 5
             orderby n descending
             select n * 2;

// Method syntax (equivalent)
var result = numbers
    .Where(n => n > 5)
    .OrderByDescending(n => n)
    .Select(n => n * 2);
```

Query syntax keywords: `from`, `where`, `select`, `orderby`, `ascending`, `descending`, `group`, `by`, `into`, `join`, `on`, `equals`, `let`.

## Filtering

```csharp
var evens = numbers.Where(n => n % 2 == 0);
var result = names.Where((name, index) => index < 3 && name.Length > 2);
var strings = mixedList.OfType<string>();
```

## Projection

```csharp
var lengths = names.Select(n => n.Length);
var anon = people.Select(p => new { p.Name, p.Age });
var indexed = names.Select((name, i) => $"{i}: {name}");
var allWords = sentences.SelectMany(s => s.Split(' '));
```

## Ordering

```csharp
var sorted = names.OrderBy(n => n);
var desc = names.OrderByDescending(n => n);
var multi = people.OrderBy(p => p.LastName).ThenBy(p => p.FirstName);
```

## Grouping

```csharp
var groups = people.GroupBy(p => p.City);
foreach (var group in groups) { /* group.Key, group items */ }

var lookup = people.ToLookup(p => p.City);
IEnumerable<Person> inNY = lookup["New York"];  // empty if missing, never throws
```

## Joining

```csharp
var result = orders.Join(customers, o => o.CustomerId, c => c.Id,
    (order, customer) => new { order.Total, customer.Name });

var grouped = customers.GroupJoin(orders, c => c.Id, o => o.CustomerId,
    (customer, orderGroup) => new { customer.Name, Orders = orderGroup });
```

## Element Operators

| Method | Returns | If empty/no match |
|--------|---------|-------------------|
| `First()` | `T` | throws |
| `FirstOrDefault()` | `T` | `default(T)` |
| `Last()` | `T` | throws |
| `LastOrDefault()` | `T` | `default(T)` |
| `Single()` | `T` | throws (also if >1) |
| `SingleOrDefault()` | `T` | `default(T)` (throws if >1) |
| `ElementAt(int)` | `T` | throws |
| `ElementAtOrDefault(int)` | `T` | `default(T)` |

## Quantifiers

```csharp
bool any = list.Any();
bool anyMatch = list.Any(x => x > 5);
bool all = list.All(x => x > 0);
bool has = list.Contains(42);
```

## Aggregation

```csharp
int count = list.Count();
int sum = list.Sum();
double avg = list.Average();
int min = list.Min();
int max = list.Max();
var minBy = people.MinBy(p => p.Age);   // .NET 6+
var maxBy = people.MaxBy(p => p.Age);   // .NET 6+
int product = numbers.Aggregate((acc, n) => acc * n);
```

## Partitioning

```csharp
var first3 = list.Take(3);
var skip3 = list.Skip(3);
var page = list.Skip(20).Take(10);
var chunk = list.Chunk(5);              // .NET 6+
var tw = list.TakeWhile(x => x < 10);
var sw = list.SkipWhile(x => x < 10);
```

## Set Operations

```csharp
var distinct = list.Distinct();
var distinctBy = list.DistinctBy(x => x.Name);  // .NET 6+
var union = list1.Union(list2);
var intersect = list1.Intersect(list2);
var except = list1.Except(list2);
```

## Concatenation and Zipping

```csharp
var concat = list1.Concat(list2);
var append = list.Append(42);
var prepend = list.Prepend(0);
var zipped = names.Zip(scores, (name, score) => $"{name}: {score}");
var tuples = names.Zip(scores);  // IEnumerable<(string, int)>
```

## Conversion

```csharp
int[] arr = query.ToArray();
List<int> list = query.ToList();
Dictionary<K,V> d = query.ToDictionary(x => x.Key, x => x.Value);
HashSet<int> set = query.ToHashSet();
var typed = arrayList.Cast<int>();
```

## Generation

```csharp
var range = Enumerable.Range(0, 10);
var repeat = Enumerable.Repeat("x", 5);
var empty = Enumerable.Empty<int>();
```

## Deferred vs Immediate

Deferred: `Where`, `Select`, `SelectMany`, `OrderBy`, `GroupBy`, `Join`, `Take`, `Skip`, `Distinct`, `Concat`, `Zip`, `OfType`, `Cast`.

Immediate: `ToArray`, `ToList`, `ToDictionary`, `ToHashSet`, `Count`, `Sum`, `Min`, `Max`, `Average`, `Aggregate`, `First`, `Single`, `Last`, `Any`, `All`, `Contains`.

## Sources
- [LINQ overview](https://learn.microsoft.com/en-us/dotnet/csharp/linq/)
- [Enumerable class](https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable)
