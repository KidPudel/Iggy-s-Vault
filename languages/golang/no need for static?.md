Go treats functions as a first-class citizens, so you don't need to wrap your functions in classes like in Java

So we can do it like this, clear and idiomatic

```go
func GetUser() *User {
	return &User{name: "", age: 0}
}
```

```go
func GetConnector() *Connector {
	return &Connector{}
}
```

[idiomatic design](https://stackoverflow.com/questions/18678135/static-method-design)
