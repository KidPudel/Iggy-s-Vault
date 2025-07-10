In golang methods has following syntax
```go
func (t LockedParameterOrOwner) FunctionName(...p Params) error {}
```

there are no such thing as objects or behavior in computer, there are only data and procedures that manipulate on that data

Basically for:
- semantics (sugar), method is belongs to/centered around specific data structure
- interfaces 


# Receiver
In golang exists a concept of a **receiver** and it's **method**.
In contrast to concept of  owner, receiver is *not* an entity and *it cannot own anything* as it is just a data type as every type (everyone is equal).
The focus is shifted on the organization here, it is an *explicit target/focus that the function is operating for*.
Instance of focused type *allowed to "receive"* function call that is being *sent* specifically/exclusively for this type and as well being captured as a parameter of a function.

# Method
**Method** - Just a function with specified receiver, which only programmatically *scopes function call to that variable*, for data organization, and then puts the receiver to function parameters.
Allowing you to structure your code in more specific/focused way.
Therefore methods are organized functions.

allowing for **fanning in** through it to in **organized way to call a function**, because the **variable IS THE RECEIVER** (making it with a signature/key/handshake to access function)!!!! and then it is just a **parameter of a functions**


**Just a way of organization through a fan-in and *NOTHING MORE***
- Scoping, but not just scoping, but rather scoping to a specific related/dependent data structure therefore **filtering** related functions, **and allowing for clean organized usage across the project.**
- Apart from that it is the same as `do_something :: proc(important_data: Data) {}`, but just with some syntax sugar

But the most important reason to have them, is it allows for implicit interfaces.

Because golang is service language, where data structures are more central and advanced part


- for what? for clean organization (which is a metadata in itself [[golang interfaces]])
- through what? through signing with a type a **receiver** contract, which allows for fan-in result. which in terms for the execution process, variable results in being **provider** for a function.


```go
 execution process ->`provider` |x| `receiver`<- func 
```


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