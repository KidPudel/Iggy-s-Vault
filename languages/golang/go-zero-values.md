# Go Zero Values

The default value automatically assigned to a variable in Go when it is declared but not explicitly initialized.

## What it does

Every type in Go has a zero value. When a variable is declared without an assignment, it holds the zero value for its type. This applies to struct fields, array elements, and variables created with `var`.

| Type                       | Zero value       |
| -------------------------- | ---------------- |
| `int`, `int8`…`int64`      | `0`              |
| `uint`, `float32/64`       | `0` / `0.0`      |
| `complex64/128`            | `0+0i`           |
| `bool`                     | `false`          |
| `string`                   | `""`             |
| `*T` (pointer)             | `nil`            |
| `func`                     | `nil`            |
| `interface{}`              | `nil`            |
| `[]T` (slice)              | `nil`            |
| `map[K]V`                  | `nil`            |
| `chan T`                   | `nil`            |
| `[n]T` (array)             | array of n zeros |
| `struct`                   | all fields zeroed|

## Sources

- https://go.dev/ref/spec#The_zero_value

## Related

- [[go-slice]]
- [[go-map]]
- [[go-array]]
- [[golang theory]]

## Process

- What is the practical difference between a nil slice and a nil map when you try to use them?
- How does Go's zero value for structs eliminate the need for explicit field initialization in many cases?
- What distinguishes a nil pointer from a zero-valued struct in Go?
