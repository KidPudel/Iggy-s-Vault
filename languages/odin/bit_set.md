Specialized type that models the [[mathematical notion of a set]]. So it is a collection of strictly specified range of valid values of specific type, where each possible values is either present or not, due to its implementation.
They are implemented as bit vectors internally for high performance, so it makes bit_set memory efficient, type safe and fast for set operations because of bitwise operations

Element type could be either an enumeration or a range (for uniqueness)
Zero value `nil` or `{}`

```odin
Direction :: enum {North, East, South, West}
Direciton_Set :: bit_set[Direction]

Char_Set :: bit_set['A'..='Z']
Number_Set :: bit_set[0..<10]

less_ten: Number_Set = {1, 2, 8}

less_ten += {9}
less_ten -= {1}

assert(9 in less_ten)
assert(1 not_in less_ten)
```


To get the number of elements set (its *cardinality*), there is `card` procedure
```odin
assert(card(less_ten) == 3)
```


Bit sets are often used to denote flags, because they are efficient, type safe and fast for set operations because of bitwise operations.

```odin
States :: enum {
	Idle,
	Walk,
	Jump,
	Run,
	Sleep,
}

State_Set :: bit_set[States]

actions: State_Set = {.Idle, .Sleep}

// morning
actions -= {.Sleep}
```