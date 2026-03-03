# Go Struct Tags

Metadata strings attached to struct fields that external Go code can read at runtime via reflection.

## What it does

A struct tag is a backtick-quoted string following a field declaration. Tags do not affect program execution directly — they are annotations read by packages like `encoding/json`, `db` drivers, and validators.

The format is `key:"value"`. Multiple tags are space-separated: `` `json:"name" db:"user_name"` ``.

The `encoding/json` package uses tags to control JSON field names and omit-empty behavior for marshaling and unmarshaling.

## Code

```go
type User struct {
    Name      string    `json:"name"`
    Password  string    `json:"password,omitempty"`
    CreatedAt time.Time `json:"createdAt"`
}

u := &User{Name: "Alice", CreatedAt: time.Now()}
out, _ := json.MarshalIndent(u, "", "  ")
// Output:
// {
//   "name": "Alice",
//   "createdAt": "..."
// }
// Password omitted (omitempty + zero value)
```

## Sources

- https://pkg.go.dev/reflect#StructTag
- https://go.dev/blog/laws-of-reflection

## Related

- [[golang theory]]
- [[go-naming-conventions]]

## Process

- How does the `encoding/json` package read struct tags at runtime?
- What does the `omitempty` option in a json struct tag do?
- What is the relationship between struct tags and Go's reflection system?
