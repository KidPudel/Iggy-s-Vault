Zig supports single-item pointer and many-item pointer

`*T`: singe-item pointer holds one item
- Dereference can be done this way `T.*`

`[*]T`: pointer to N items
- Access is done with `ptr[i]`
- [[zig slice]] syntax
- pointer arithmetic


`*[N]T`: Close to the [[Array]], pointer to the N-items
`[]T`: Is a [[zig slice]], a [[fat pointer]], which contains a pointer to the `[*]T`


To get a singe-item pointer use ``