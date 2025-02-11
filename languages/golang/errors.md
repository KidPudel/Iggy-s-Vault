In golang philosophy for the errors is that errors are data as well, and therefore you can interact with it
In golang it is idiomatic to manually and explicitly manage your errors, so that every case in your program is covered, and if error happens you know exactly where that happened

Golang's design lets you always take into account scenarios of the error

it is designed to pass the error down the stream **decorating** it to track each place from inner to outer.

In golang there are explicit absence of the value, so it is something or nil, in this case error or nil.


Using `errors.Is` in Go is better than using `==` for error comparison because:

1. **Error Wrapping Support**: `errors.Is` checks if an error is in the chain of wrapped errors (using `fmt.Errorf("%w", err)` or `errors.Join`), while `==` only works for direct equality.
    
2. **More Robust Error Handling**: Since Go encourages error wrapping, `errors.Is` ensures that comparisons remain valid even if errors are wrapped, avoiding brittle checks.
    
3. **Consistency with Standard Library**: Many standard library functions wrap errors, making `errors.Is` the idiomatic way to check for specific error types.

error.Is(err, target error) bool -> compare errors wrapped
error.As(err error, target any) bool -> compare with custom errors wrapped