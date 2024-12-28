type assertion is a way to check the underlying type of the variable
it is done through .() checking

or with panic if value is not the type of it
```odin
s := value.(string)
```
or
```odin
s, ok := value.(string)
```


# Type switch statement
```odin
switch v in value {
case string:
	#assert(type_of(v) == string)
case bool:
	#assert(type_of(v) == bool)
}
```


# union [[tags]]
`#no_nil` states it does not have a nil value.
Those unions must have at least 2 variants and the first variant is its default type
```odin
Value :: union #no_nil {bool, string}
```

`#shared_nil` requires all variants to have nil values

`#align` like structures
```odin
union #align(4) {...} //align to 4 bytes
```