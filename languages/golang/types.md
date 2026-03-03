# Go Types

https://go.dev/ref/spec#Types

Basic: `bool`, `string`, `int`, `int8/16/32/64`, `uint`, `float32/64`, `complex64/128`
Aliases: `byte` = `uint8`, `rune` = `int32` (Unicode code point)

```go
type newInt int         // new distinct type — requires explicit conversion
type newFloat = float64 // type alias — interchangeable, no conversion needed
```
