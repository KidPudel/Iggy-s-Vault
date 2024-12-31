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


In C unions are structures, but all members occupy the same memory, so when you write to one, all members gets overwritten


There are 2 main use cases
1. Storing a bunch of different types and you don't know what type exactly it will be at runtime
2. Type punning, to represent something usual in unusual format


i have a feeling that the difference between [[enum]] and union approaches is for using unions when types are too different, like "Thing In the Pocket", it could be a pen or it could be a gun, where as for enum, it is just for slight data inside instance difference.
So [[enum]] for *shallow difference*
where as unions are for *deep difference*

- union approach, since it is self containing variants is suited for cases where there is a possibility for extensions and **growth of variants of types**
- [[enum]]erated [[dynamic arrays]] approach is more memory efficient and for smaller and simpler cases **where you have one type with different data** so it is for one purpose like list of different sounds for different events