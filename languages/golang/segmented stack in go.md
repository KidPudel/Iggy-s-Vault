In Golang [[stack]] is segmented
Segmented stack is a technique where instead of allocating a large, fixed-size stack for each goroutine, the stack is split into smaller *linked* ([[linked list]]) segments instead of one contiguous block of memory.
This lets the stack grow and shrink as needed, yet still be limited in its potential

It's kind of like [[slices under the hood]]

