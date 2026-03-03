# Go HTTP Server

`Handler` interface: implement `ServeHTTP(w, r)`. `ServeMux` routes patterns to handlers.

Design: group handlers by resource, not by HTTP method. Share expensive instances (DB, logger) via package-level vars — same instance across all handlers.

```go
type WishesHandler struct{ db *DB }

func (h WishesHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    switch r.Method {
    case http.MethodGet:
    case http.MethodPost:
    }
}

mux := http.NewServeMux()
mux.Handle("/wishes", WishesHandler{db: db})
log.Fatal(http.ListenAndServe(":8080", mux))
```

For more control: instantiate `http.Server` directly to set timeouts.

https://pkg.go.dev/net/http
https://pkg.go.dev/net/http#ServeMux
