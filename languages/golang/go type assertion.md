```go
func main() {
    n := 12
    var x interface{} = n // Assign n to an interface{} type variable

    if num, ok := x.(int); ok {
        fmt.Printf("x is an int with value %d\n", num)
    } else {
        fmt.Println("x is not an int")
    }
}

```