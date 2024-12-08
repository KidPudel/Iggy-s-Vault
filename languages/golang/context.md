Context is a mechanism for carrying:
- deadlines: set maximum time for the process of the request
- cancellation: way of terminating request
- request-scoped values: data associated with a request, passed *down the call stack*, passing  to the down nested function, middleware, or actual request handler

# how to use

> NOTE: key, must not be be a string or any build in types, to avoid collisions with packages using context

```go
type SgContextKey string
```

setting value, on base of `context.Backend()`, never cancellable, empty context
```go
key := SgContextKey("test")
ctx := context.WithValue(context.Background(), key, "value")
```

updating/modifying  context
```
req.WithContext(ctx)
```


getting value 
```go
if value := ctx.Value(key); value != nil {
	fmt.Println(value) // "value"
}
```



```go
package main

import (
	"context"
	"fmt"
	"net/http"
)

type AuthInfo struct {
	// Define your authentication information fields here
	Token   string
	IsLoggedIn bool
	// Other fields...
}

func authenticateMiddleware(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		// Perform authentication logic, e.g., validate JWT token
		token := r.Header.Get("Authorization")

		// Create AuthInfo instance and store it in the request context
		authInfo := &AuthInfo{
			Token:      token,
			IsLoggedIn: /* Your authentication logic */,
			// Set other fields...
		}

		ctx := context.WithValue(r.Context(), "AuthInfo", authInfo)
		r = r.WithContext(ctx)

		// Call the next handler in the chain
		next.ServeHTTP(w, r)
	})
}

func handler(w http.ResponseWriter, r *http.Request) {
	// Retrieve AuthInfo from the request context
	authInfo, ok := r.Context().Value("AuthInfo").(*AuthInfo)
	if !ok {
		http.Error(w, "Authentication information not found", http.StatusUnauthorized)
		return
	}

	// Access authentication information
	fmt.Printf("Token: %s, IsLoggedIn: %t\n", authInfo.Token, authInfo.IsLoggedIn)

	// Handle the request
	w.Write([]byte("Hello, world!"))
}

func main() {
	// Attach the authentication middleware
	http.Handle("/", authenticateMiddleware(http.HandlerFunc(handler)))

	// Start the server
	http.ListenAndServe(":8080", nil)
}
```


# Cancellation
```go
ctx, cancel := context.WithCancel(context.Background())

for {
	go worker(ctx)
}
cancel()
```
`
```go
func worker(ctx context.Context, in <-chan int) {
	for n := range in {
		select {
			case out <- n:
			case <-ctx.Done():
				return
		}
	}
}
```