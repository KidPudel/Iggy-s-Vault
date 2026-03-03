# C# Namespaces

## File-Scoped (C# 10+)

```csharp
namespace MyApp.Models;

class Player { }
class Enemy { }
// all types in this file belong to MyApp.Models
```

## Block-Scoped (traditional)

```csharp
namespace MyApp.Models
{
    class Player { }
    class Enemy { }
}
```

## Global Using (C# 10+)

```csharp
global using System;
global using static System.Math;
global using MyAlias = MyApp.Models.Player;
```

## Using Statements

```csharp
using System.Collections.Generic;
using static System.Console;        // import static members
using Alias = System.Collections.Generic.Dictionary<string, int>;
```

## Sources
- [Namespaces](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/namespaces)
