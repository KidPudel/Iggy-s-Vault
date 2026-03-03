# Why Goroutines Are Lightweight

- Start with a small stack (~2–8 KB) that grows and shrinks as needed
- Runtime-managed — not mapped 1:1 to OS threads
- Context switch happens in user space, no kernel involvement
- Cooperative scheduling at channel ops and function calls

https://go.dev/doc/faq#goroutines
