In golang methods has following syntax
```go
func (t LockedParameterOrOwner) FunctionName(...p Params) error {}
```

Function makes a method a non negotiable/**locked** (different from necessary) parameter that acts as the owner and also makes it exclusive to that type  