# Type Assertion

Extracts the concrete value from an interface. Safe two-value form returns `(value, ok)` — use this to avoid panics.

https://go.dev/ref/spec#Type_assertions

```go
if num, ok := x.(int); ok {
	fmt.Printf("x is an int: %d\n", num)
}

// type switch
switch v := x.(type) {
case int:
case string:
}
```
