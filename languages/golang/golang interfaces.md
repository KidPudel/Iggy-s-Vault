In Go there is a lot of times where we have an [[interface]] that does something.
For example `Handler` from `net/http` that is the second parameter in `ListenAndServe` function, where `Handler` does indeed serve (handles), but it *maybe* does not do/has everything inside it that we want.
Go's philosophy is intend to implement those base blocks to enhance the functionality and content, because of that it is so easy to implement interfaces in Go.  

If you know that you are going to have custom version of the default one -> this is an interface, and you need to implement it

Or many versions that at the base basically the same, use interfaces

## wrapping
> ***NOTE:*** if some function takes an interface, and some type implements it, (for example `ServeMux` that implements `Handler`),  with some functionality, we can wrap with another custom implementation of interfaces

```go
type Connecter interface {
	Connect(timeout int) bool
}
```

# under the hood
Interface contains of 2 pieces.
1. Type information: **metadata about the concrete type** of the value stored in the interface, like size of a string, alignment in memory, kind and so on
2. Data pointer: pointer to the data

```go
type error interface {
	TypeInfo
	DataPointer unsafe.Pointer
}
```

Thats why `nil` pointer is not the same as `nil`


# Detect type
or type assertion with `v, ok := i.(string)`
or with [[reflection]]

```go
func main() {
    x := 42
    t := reflect.TypeOf(x) тоже для типов по сути
    fmt.Println("Type:", t)       // Type: int
    fmt.Println("Kind:", t.Kind()) // Kind: int
}

```



```go

func printErr(err error) {
	if err != nil {
		fmt.Println("error: ", err)
	}
}

func main() {
	err := *MyError
	printErr(err) // err doesnt equals err, but its value is nil pointer
	// so "error: nil"
}
```