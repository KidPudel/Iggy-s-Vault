[Go REST](https://www.jetbrains.com/guide/go/tutorials/rest_api_series/stdlib/)
### Default implementations

`http` already has the default implementations for handlers and servers like shown below:
```go
func main() {

http.HandleFunc("/page2", Page2)
  http.HandleFunc("/", Index)
  http.ListenAndServe(":3000", nil)
}
```

```go
func main() {
  mux := http.NewServeMux()

  mux.HandleFunc("/page2", Page2)
  mux.HandleFunc("/", Index)
  http.ListenAndServe(":3000", mux)
}
```


---
# Server

[`ListenAndServer`]() starts an HTTP server with a given address and handler
> NOTE: hander is typically a nil, which means [DefaultServeMux](https://pkg.go.dev/net/http#DefaultServeMux). [Handle](https://pkg.go.dev/net/http#Handle) and [HandleFunc](https://pkg.go.dev/net/http#HandleFunc) add handlers to [DefaultServeMux](https://pkg.go.dev/net/http#DefaultServeMux)


```go
http.Handle("/foo", fooHandler)

http.HandleFunc("/bar", func(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Hello, %q", html.EscapeString(r.URL.Path))
})

log.Fatal(http.ListenAndServe(":8080", nil))
```

## to have more control, we can instantiate our own Server
```go
s := &http.Server{
	Addr:           ":8080",
	Handler:        myHandler,
	ReadTimeout:    10 * time.Second,
	WriteTimeout:   10 * time.Second,
	MaxHeaderBytes: 1 << 20,
}
log.Fatal(s.ListenAndServe())
```

---
# Handler
`Handler` *handles* HTTP requests that is directed via router. It's an interface that declares `ServeHTTP` function that handles requests.

We can implement our custom type to have a custom dependencies (like [[std database]] instance),  that will implement `Handler`, and navigate with [[ServeMux]] using `Handle()` function with related routes (like GET `getItems`, POST `getItems`, or if the request is simple, then we can use `HandleFunc()`
As in [[go design philosophy]].

> **DESIGN:** We can divide handlers by the categories, to ***reduce the amount handlers***, but ***yet not make a big handlers***, and also important, utilising the singletons with [[singleton pattern]] to allow use of the ***same instances across different handlers***


---
[[simple server example]]
# Routing
[[ServeMux]] is the standard router
## Patterns
https://pkg.go.dev/net/http#hdr-Patterns
Patterns can match the method, host and path of a request. Some examples:
- "/index.html" matches the path "/index.html" for any host and method.
- "GET /static/" matches a GET request whose path begins with "/static/".
- "example.com/" matches any request to the host "example.com".
- "example.com/{$}" matches requests with host "example.com" and path "/".
- "/b/{bucket}/o/{objectname...}" matches paths whose first segment is "b" and whose third segment is "o". The name "bucket" denotes the second segment and "objectname" denotes the remainder of the path.
