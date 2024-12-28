Enumeration types define a new type whose values consist of the ones specified, they are also ordered
```odin
Foo :: enum {
	A,
	B = 5,
	C = 12,
	D = 133,
}
```


# backing type also could be specified
```odin
Foo :: enum u8 { A, B, C }
```

# Implicit selector expression
Abbreviated way to access a member of enum, in a context where type inference can determine the implied type
```odin
switch f {
	case .A:
	case .B:
	case .C:
}
```


# using, to bring to the current scope
```odin
main :: proc() {
	F :: enum {A, B, C}
	using F
	a := A
}
```


# Iteration
```odin
Direction :: enum{North, East, South, West}
for direction, index in Direction {
	fmt.println(index, direction)
}
```


# Enumerated array
Allows to use Enum values as indexes

```odin
Direction_Vectors :: [Direction]Vector {
	.North = { 0, -1 },
	.East = { +1, 0 },
	.South = { 0, +1 },
	.West = { -1, 0 },
}
```