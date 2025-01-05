structure of array is a way of aligning data in a columnar way, like in [[columnar database]]

```odin
Foo :: struct {a, b: f32}

// Stored as 'abababab' in memory
s1: [4]Foo

// Stored as 'aaaabbbb' in memory
s2: #soa[4]Foo
```

[[tags]]