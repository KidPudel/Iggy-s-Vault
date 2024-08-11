If indicated in function parameters like so:
```go
func (p Person) Greet(who ...string) {
	fmt.Println("Hello ", who, "!", "/nI'm", p.Name)
}
```

and to pass a slice to it
```go
person := Person{"Iggy"}
peopleToGreat := []string{"Maya", "Luna", "Stacy"}
person.Great(peopleToGreat...)
```