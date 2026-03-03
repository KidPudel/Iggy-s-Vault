# Type Alias

Alternative name for an existing type — fully interchangeable, not a new distinct type.

https://go.dev/ref/spec#Type_declarations

```go
type MetadataAlias = mo.Option[Metadata] // alias — same type, no conversion needed
type MyInt int                            // new distinct type — requires explicit conversion
```
