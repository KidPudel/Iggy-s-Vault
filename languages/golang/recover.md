If the panic is happens, then recover function could catch that panic and convert into an error
```go
defer func(){
	if recover() != nil {
		err = errors.New("got panic durring execution, most likely reading empty page")
	}
}()
```