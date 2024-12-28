In Odin union is a data structure that contains in itself a collection of types it could be
Zero value of union is `nil`
```odin
Value :: union {
	bool,
	int32,
	f32,
	string,
}

v: Value = "hello"

s, isString := v.(string) // true
```