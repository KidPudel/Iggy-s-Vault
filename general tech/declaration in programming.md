Statement that announces the fact of existence of entity (class, variable, function) to the [[compiler]] without providing implementation nor \*allocating a memory (bringing to existence), because this is the job of [[definition in programming]]

It is a subpart of the [[definition in programming]]

> [!quote] "Hey, just so you know, there is a thing named `x` and it is an `integer`. I promise it exists somewhere. Trust me for now."

Role: The ID card
Job: "I exists and here is my name"

1. Introduces name/identifier into scope
2. Tells what this entity at its core (function, class) -> how to use it (e.g. "use like a function")
3. \* Allocating can happen internally in cases of declaring a variables (`var number: int` will allocate memory and set the default value to 0, which is basically a definition) ([[definition in programming]])

Not to confuse with [[declarative style]] paradigm


> [!note] A declaration stops being just a declaration the moment it causes the compiler to reserve storage (memory). At that point, it becomes a *[[definition in programming]]*.

