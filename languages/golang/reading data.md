# IO Package Guide

Which package for what:
- `os` — filesystem and process I/O (`os.Open`, `os.Stdin`)
- `io` — core interfaces (`io.Reader`, `io.Writer`) and utilities (`io.Copy`)
- `bufio` — buffered reading + `Scanner` for line-by-line
- `bytes` — operations on `[]byte`, mirrors `strings`
- `fmt` — formatted scan/print (`fmt.Fscan`, `fmt.Fprintf`)

```go
scanner := bufio.NewScanner(file)
for scanner.Scan() {
    fmt.Println(scanner.Text())
}

reader := bufio.NewReader(os.Stdin)
line, _ := reader.ReadString('\n')
```

https://pkg.go.dev/bufio
https://pkg.go.dev/io
