
[[structure]] can define [[metadata]], when you read data from database or APIs, to control how the data is assigned to the fields of the struct.
You can do this by utilising ***struct tags***

Struct tags are the small pieces of information that are assigned to the fields of the struct, that provides the instruction to other Go code that works with this struct.

# In Go

```go
type Person struct {
	name string `example:name`
}
```

Other Go code then can be able to extract value from the key it requires.

> NOTE: tags does not effect the execution of the code
```go
package main

import "fmt"

type User struct {
	Name string `example:"name"`
}

func (u *User) String() string {
	return fmt.Sprintf("Hi! My name is %s", u.Name)
}

func main() {
	u := &User{
		Name: "Sammy",
	}

	fmt.Println(u)
}
```

```bash
OutputHi!
My name is Sammy
```



# Encoding JSON example

The standard package `encoding/json` makes use of tags as annotations indication to the encoder how would you like to name your fields in JSON format
> NOTE: it works both ways! (*encoding and decoding*)

## Example without use of tags
```go
package main

import (
	"encoding/json"
	"fmt"
	"log"
	"os"
	"time"
)

type User struct {
	Name          string
	Password      string
	PreferredFish []string
	CreatedAt     time.Time
}

func main() {
	u := &User{
		Name:      "Sammy the Shark",
		Password:  "fisharegreat",
		CreatedAt: time.Now(),
	}

	out, err := json.MarshalIndent(u, "", "  ")
	if err != nil {
		log.Println(err)
		os.Exit(1)
	}

	fmt.Println(string(out))
}
```

```
Output{
  "Name": "Sammy the Shark",
  "Password": "fisharegreat",
  "CreatedAt": "2019-09-23T15:50:01.203059-04:00"
}
```

### Problem
JSON naming conventions are with a snake_case or camelCase, but Go exposes fields (makes it public), when they are called with a capital letter

## Example with struct tags

```go
package main

import (
	"encoding/json"
	"fmt"
	"log"
	"os"
	"time"
)

type User struct {
	Name          string    `json:"name"`
	Password      string    `json:"password"`
	PreferredFish []string  `json:"preferredFish"`
	CreatedAt     time.Time `json:"createdAt"`
}

func main() {
	u := &User{
		Name:      "Sammy the Shark",
		Password:  "fisharegreat",
		CreatedAt: time.Now(),
	}

	out, err := json.MarshalIndent(u, "", "  ")
	if err != nil {
		log.Println(err)
		os.Exit(1)
	}

	fmt.Println(string(out))
}
```

```
Output{
  "name": "Sammy the Shark",
  "password": "fisharegreat",
  "preferredFish": null,
  "createdAt": "2019-09-23T18:16:17.57739-04:00"
}
```

