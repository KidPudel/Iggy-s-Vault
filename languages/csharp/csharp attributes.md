# C# Attributes

```csharp
[Obsolete("Use NewMethod instead")]
[Obsolete("Removed", error: true)]       // compile error
void OldMethod() { }

[Conditional("DEBUG")]                    // call removed if symbol not defined
void DebugLog(string msg) { }

// Caller info (auto-filled by compiler)
void Log(string msg,
    [CallerMemberName] string member = "",
    [CallerFilePath] string file = "",
    [CallerLineNumber] int line = 0) { }

[Flags]
enum Permissions { Read = 1, Write = 2, Execute = 4 }

[Serializable]
[field: NonSerialized]    // target the backing field

[AttributeUsage(AttributeTargets.Class | AttributeTargets.Method, AllowMultiple = false)]
class MyAttribute : Attribute { }

// Assembly-level
[assembly: InternalsVisibleTo("TestProject")]
```

## Sources
- [Attributes](https://learn.microsoft.com/en-us/dotnet/csharp/advanced-topics/reflection-and-attributes/)
