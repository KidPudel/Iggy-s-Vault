# Methods

There are no objects in a computer — only data and procedures that manipulate data.

A **receiver** is not an entity. It's an explicit target type — the focus of the function. The instance *receives* the call, and is just a parameter under the hood.

A **method** is a function with a receiver bolted on. Organization tool, not a behavior model. Allows fan-in: variable → receiver contract → function access.

```go
func (g *Game) Update(status int) {}
// same as:
func GameUpdate(g *Game, status int) {}
```

Most important use: enables implicit interface satisfaction. No `implements` — if your type has the methods, it satisfies the interface.

https://go.dev/doc/effective_go#methods
