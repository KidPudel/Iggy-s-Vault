#backend
# database

### flow
1. create a **wish list**
2. add **wish** to list
3. add permission for users to view your lists

### Relationship
we have *one* list to store *many* wishes
>*one to many*

>*therefore many "wish"es can associate itself with one of the "wish list"'s*

| Wishes |
| ---- |
| id |
| name |
| rate |
| wish_list_id |

| Wish Lists |
| ---- |
| id |
| name |

---

*one* list can be viewed by *many* people 
*one* person can view *many* lists
>*many to many*

>*this avoid problem with uniqueness requirement*


| Permissions |
| ---- |
| list_id |
| tg_user_id |


# api

Go in its standard library provides a package [net/](https://pkg.go.dev/net@go1.22.0#section-documentation)  that implements interface for network I/O, inluding TCP/IP, UDP and domain name resolution, and Unix domain socket.

`net/` package has its own packages that we will use to implement the necessary functionality, like [http/](https://pkg.go.dev/net/http@go1.22.0) that provides HTTP and server implementation

To test the package, first, implement:
1. [x] api call async
```go
package main

import (
	"encoding/json"
	"fmt"
	"net/http"
)

func main() {
	ch := make(chan map[string]any)
	go apiCall(ch)

	for n := range 10 {
		fmt.Println("do some for me ", n)
	}

	response := <-ch

	fmt.Println(response["dept"])
}

func apiCall(ch chan map[string]any) {
	response, err := http.Get("https://api.supergood.ru/getitems.php")
	if err != nil {
		ch <- nil
	}
	defer response.Body.Close()

	var jsonData interface{}
	err = json.NewDecoder(response.Body).Decode(&jsonData)
	if err != nil {
		ch <- nil
	}
	data, ok := jsonData.(map[string]any)
	if ok {
		ch <- data
	} else {
		ch <- nil
	}
}


```
2. [ ] api endpoints


