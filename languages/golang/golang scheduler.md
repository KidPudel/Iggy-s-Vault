Part of a Go [[runtime]], scheduler is a program or more of a **part of the program (runtime)**, so it's a subsystem code implemented with Go and assembly for performance-critical parts.
Analogy would be a factory's operating system manager

It is **responsible for distributing** (managing which, when, and where (on which thread) goroutine should run) goroutines across the OS threads to run on.

Scheduler utilizes CPU cores and distributing goroutines across them for maximum efficiency.
So if CPU has 1 core, then scheduler will rapidly switch between them giving it a slice of CPU time

[[GMP]]