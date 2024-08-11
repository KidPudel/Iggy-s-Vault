```go
type Animalizer interface {
	MakeSound()
}

type Animal struct {
}

func (animal Animal) MakeSound() {
}

type Dog struct {
	Animal // anonymous type
}


func PrintAnimal(animal *Animalizer) {

}
```

here `Dog` implements an interface by having inside of it type that implements it