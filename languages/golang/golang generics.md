Added in 1.18

3 main features:
1. Type parameters (with constraints)
2. Type inference
3. Type set

# Type parameters
```go
func min[T any](a T, b T) T {
	if a > b { // won't work, see below
		return a
	}
	return b
}

func main() {
	c := min[int](1, 2)
	d := min[float64](2.3, 5.65)
}
```

We register a local generic type to our function in `[]` passing in name and **constraint**
> *constraint*: indicating available types to the generic type
> But using ***more general types, limits their features***, that's why any not always a good idea

# Type inference
Lets infer the type provided in generic function, so that compiler will figure out by itself
```go
c = min(2, 3)
```


# Underlying type
if we pass a custom type that implements a type that is allowed in generic function, this will throw an error

```go
type superInt int
var i superInt = 3

func min[T int](a T, b T) T {
	return a + b
}

func main() {
	c := min(i, 5) // superInt does not satisfy int
}
```

To actually allow all types that are underlying int, we use `~`
```go
type superInt int
const i superInt = 3

func min[T ~int](a T, b T) T {
	return a + b
}

func main() {
	c := min(i, 5) // ok
}
```


# Type set
When we want to expand the ***range of acceptance of our generic type***, we can define a set of possible types

```go
type superNumber interface {
	~int, ~float64, ~float32
}

func min[N superNumber](a N, b N) N {
	if a < b {
		return a
	}
	return b
}
```

also we can define a generic type set right in a function generic register

```go
func min[T ~int | ~float64 | ~float32](a T, b T) T {
	if a < b {
		return a
	}
	return b
}
```


>NOTE: there is also a `comparable` type