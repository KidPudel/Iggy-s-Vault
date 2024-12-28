In golang methods has following syntax
```go
func (t LockedParameterOrOwner) FunctionName(...p Params) error {}
```

there are no such thing as objects or behavior in computer, there are only data and procedures/functions
But it is nice to define some *boundaries*, when there are some "exclusive" procedures happening with one data structure, that acts as pivot.
So this is a way of associating procedurals for some specific data structure.

To define outline for the procedures we can achieve it by attaching non negotiable/**locked** (different from necessary) parameter that acts as the owner data structure/type and also makes it exclusive to that type. 

We basically say, that this must-have data structure must be filled with data, for the function to work properly.
Now this is just non-optional function parameters, just like in odin.
```odin
game_update :: proc(game: ^Game, newStatus: i32) {}
```

But what locking to specific data structure does, is to helps **define a path** that first, we need to establish/fill data, as well as **defining the scopes** more flexibly
```go
game := NewGame(ctx)
game.update(1)
----

func (g Game) update(newStatus int) {}
```