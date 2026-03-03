# C# Nullable Types

## Nullable Value Types (`T?`)

```csharp
int? x = null;
int? y = 42;

y.HasValue   // true
y.Value      // 42 (throws if null)
y.GetValueOrDefault()    // 42
y.GetValueOrDefault(10)  // 42

int z = y ?? 0;          // 42
```

## Null-Conditional Operator (`?.` and `?[]`)

```csharp
string? name = person?.Name;
int? len = person?.Name?.Length;
char? c = arr?[0];
person?.DoSomething();
```

## Null-Coalescing Operator (`??`)

```csharp
string result = input ?? "default";
int length = text?.Length ?? 0;
```

## Null-Coalescing Assignment (`??=`)

```csharp
list ??= new List<int>();  // assigns only if list is null
```

## Null-Forgiving Operator (`!`)

```csharp
string s = maybeNull!;  // suppresses nullable warning
```

## Sources
- [Nullable value types](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/nullable-value-types)
- [Nullable reference types](https://learn.microsoft.com/en-us/dotnet/csharp/nullable-references)
