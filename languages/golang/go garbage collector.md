# Go Garbage Collector

Concurrent tri-color mark-and-sweep GC. Runs concurrently with the program to minimize stop-the-world pauses. Triggered when heap reaches a size threshold.

https://go.dev/doc/gc-guide
