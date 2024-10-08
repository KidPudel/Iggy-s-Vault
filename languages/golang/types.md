### basic
- `bool`
- `string`
- `int`, if os 64x => `int64` (int8,16,32,64)
- `uint`, the same as int, but additionaly `uintptr`
- `float32`, `float64`
- `complex64`, `complex128`

### aliases
- `byte`, is for `uint8`
- `rune`, is for `int32` that represents _unicode code point_


## Define your own types
To define a new type:
1. just start with `type`
2. give it a name `foo`
3. specify a type on which a new type is going to be ***based***
> SUMMARY: blueprint, define a type, by giving it's name and what it is going to be at a base
```go
type newInt int

type person struct {
  name string
  age int
}

type (
  newString string // type definition
  newFloat = float64 // type aleas (doen't create a new type and doesn't need a conversion)
)
```

# Type methods
To define a method for the type, we create a function with the ***receiver*** specified, indicating *what type is going to access it*.
This defines methods/what it can do that ***belongs*** to the Person type ***instance***, basically it's just a semantics
```go
type Person struct {
	name string
}

func (p Person) Introduce() {
	fmt.Println("Hello, I'm ", p.name)
}
```


