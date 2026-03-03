# Go Errors

Errors are values. Pass them, decorate them, check them explicitly — don't throw them.

Idiomatic: handle every error at the call site. Wrap with context as you propagate up so the failure chain is traceable.

```go
return fmt.Errorf("service.GetUser: %w", err) // wrap

errors.Is(err, target)  // checks full chain — use instead of ==
errors.As(err, &target) // extracts concrete type from chain
```

https://go.dev/blog/errors-are-values
https://pkg.go.dev/errors
