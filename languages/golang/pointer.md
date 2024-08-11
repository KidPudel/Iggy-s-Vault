> In Go pointers like in C, but without arithmetics for safety.

When declaring a pointer `*T` of type `T` we allocate a data in heap, so that we can use it everywhere and any time
So in go, if we don't assign point, its value is `nil`

_pointer to the int value_
```go
var p *int
```

_`&` generates a pointer (returning address, where we can point later)_
```go
a := 1
p = &a
```

_`*` denotes (points to) pointer's underlying value_
```go
fmt.Println(p) // 0x000121 - some address
fmt.Println(*p) // 1
```

_when pointing to (denoting) value that stored in address, we basically accessing its value, so we can change it_
```go
*p = 10
fmt.Println(a) // 10
```


Basically we pass something by reference, to not copy it, in order to *point* to the actual variable, and if we manipulate on it by to changing/altering **it**, 


```go
package main  
  
import (  
    "encoding/json"  
    "fmt"    "log")  
  
type Animal struct {  
    Name string  
}  
  
func (a *Animal) ChangeName() {  
    a.Name = "New name"  
}  
  
type Person struct {  
    Name string `protobuf:"bytes,1,opt,Name=requestID,proto3" json:"name"`  
    Age  int    `protobuf:"bytes,1,opt,Name=requestID,proto3" json:"age"`  
    Pet  Animal `protobuf:"bytes,1,opt,name=requestID,proto3" json:"pet,omitempty"`  
}  
  
func (p Person) ChangeAge() {  
    p.Age = 20  
    p.Pet.ChangeName() // won't affect global, becase ChangeAge took receiver by value
} 

func (p *Person) ChangeAgeWithRef() {  
    p.Age = 20  
    p.Pet.ChangeName() // will affect global, because the "ChangeAgeWithRef" receiver grabbed by reference
}  
  
func main() {  
    person := Person{  
       Name: "john",  
       Age:  10,  
       Pet: Animal{  
          Name: "Morgan",  
       },  
    }  
  
    person.ChangeAge()  
  
    rawPerson, err := json.Marshal(person)  
    if err != nil {  
       log.Fatal(err)  
    }  
    fmt.Println(string(rawPerson))  
}
```

we can pass references to safe or share the object

[PR]([BE] Добавление ссылочного чека в сервис "Чек")