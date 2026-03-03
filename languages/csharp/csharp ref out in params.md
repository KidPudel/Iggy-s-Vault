# C# ref, out, in, params

## ref

Passes by reference. Must be initialized before passing.

```csharp
void Increment(ref int value) { value++; }

int x = 5;
Increment(ref x);  // x is now 6
```

## out

Passes by reference. Must be assigned inside the method.

```csharp
bool TryParse(string s, out int result) { ... }

if (int.TryParse("42", out int result))
{
    // result = 42
}

// Discard
int.TryParse("42", out _);
```

## in

Passes by read-only reference. Cannot be modified inside the method.

```csharp
void Print(in Vector3 position) { /* cannot modify position */ }

Print(in myVector);  // explicit
Print(myVector);     // 'in' is optional at call site
```

## params

Variable-length argument list. Must be last parameter.

```csharp
void Log(string format, params object[] args) { }
Log("Values: {0}, {1}", 10, 20);
Log("No args");

// params with Span (C# 13+)
void Process(params ReadOnlySpan<int> values) { }
```

## Optional Parameters and Named Arguments

```csharp
void Draw(int x, int y, string color = "red", float opacity = 1f) { }

Draw(10, 20);
Draw(10, 20, opacity: 0.5f);
Draw(x: 10, y: 20);
```

## Sources
- [Method parameters](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/method-parameters)
- [ref keyword](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/ref)
- [out keyword](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/out-parameter-modifier)
- [in keyword](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/in-parameter-modifier)
- [params keyword](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/params)
