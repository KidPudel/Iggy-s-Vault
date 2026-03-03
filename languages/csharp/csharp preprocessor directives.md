# C# Preprocessor Directives

```csharp
#if DEBUG
    // compiled only in Debug
#elif UNITY_EDITOR
    // compiled only in Unity editor
#else
    // otherwise
#endif

#define MY_SYMBOL
#undef MY_SYMBOL

#region My Section
// collapsible code block
#endregion

#warning This is a build warning
#error This is a build error

#pragma warning disable CS0168    // suppress specific warning
int unused;
#pragma warning restore CS0168

#nullable enable
#nullable disable
#nullable restore

#line 200 "SpecialFile.cs"        // override line/file info
#line default                     // restore
```

## Sources
- [Preprocessor directives](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/preprocessor-directives)
