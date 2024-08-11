`iota` in Go's way of creating successive (enumerable) integer 1, 2, 3... like `enum`

```go
// optional ideomatic/contextual integer
type Direction int
const (
	North Direction = iota
	South
	West
	East
)
```