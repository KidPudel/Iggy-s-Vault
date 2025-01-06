In Odin union is a data structure that contains in itself a collection of types it could be.
which allows for flexible *fan-out* (like in [[concurrency patterns]]) data structure capabilities (meaning, that it can change its **data capabilities**)
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


structures, but instead of fields being aligned in memory, all members occupy the same memory, so when you write to one, all members gets overwritten


There are 2 main use cases
1. Storing a bunch of ***different*** types and you don't know what type exactly it will be at runtime and you can check it to act accordingly. in other words [[polymorphism]]
2. Type punning, to represent something usual in unusual format


# difference between [[enum]] and `union` approaches:
- `enum` is used to indicate the different ***hierarchical** data indication*, so it is **the data**.
- `union` is the fundamental change in **data structure** that is a ***shape-shifter*** by its nature. which allows for flexible *fan-out* (like in [[concurrency patterns]]) data structure capabilities (meaning, that it can change its **data capabilities**)

- union approach, since it is self containing variants is suited for cases where there is a possibility for extensions and **growth of variants of types**

// hierarchical approach
- [[enum]]erated [[dynamic arrays]] approach is more memory efficient and for smaller and simpler cases **where you have one type with different data indicating something** so it is for **one purpose**, that allows to make it strict and safe from disallowed usages. like list of different sounds for different events