Type alias is an alternative name for an existing type, it is used for a **reference** or **compatibility**, so alias and type are fully interchangeable

```go
type MetadataAlias = mo.Option[Metadata]
```
> NOTE: type alias does not create a new distinct/separate type, it is a reference

if you want to create a separate type just use
```go
type NewType OldType
type MyInt int
```