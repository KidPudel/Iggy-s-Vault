> A property of a universal computation - Turing completeness - is that a computer program can write a computer program.

But code need to be integrated into the build process so their output can be compiled.
It could be easy to do with external tool like [[Make]]. But in Go, where go tool gets all necessary information, we can do it with `go generate`
like `go test`, it scans all packages to be tested, `go generate` looks for comments `//go:generate <command>` and generates a code