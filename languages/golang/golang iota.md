# iota

Auto-incrementing integer counter in `const` blocks, reset to 0 at each new `const` block.

https://go.dev/ref/spec#Iota

```go
type Direction int
const (
	North Direction = iota // 0
	South                  // 1
	West                   // 2
	East                   // 3
)
```
