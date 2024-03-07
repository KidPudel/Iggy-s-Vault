# main

```go
package main

import (
	"fmt"
	"log"
	"net/http"

	"github.com/KidPudel/learn-go-api/api"
)

func main() {
	// router handler
	mux := http.NewServeMux()

	mux.HandleFunc("/simple", func(response http.ResponseWriter, request *http.Request) {
		fmt.Fprintln(response, "simple handler, that does not contain anything.")
	})

	mux.Handle("/getWishes", api.WishesHandler{})

	log.Fatal(http.ListenAndServe(":8081", mux))
}
```

# db

```go
package db

import (
	"fmt"

	"github.com/jackc/pgx"
)

type DB struct {
	ConnnectionPool *pgx.ConnPool
}

var db *DB


func (db *DB) Close() {
	defer db.ConnnectionPool.Close()
}

func ConnectDB() (*DB, error) {
	// singleton!!!
	if db != nil {
		return db, nil
	}
	config := pgx.ConnConfig{
		Host:     "localhost",
		Port:     5432,
		Database: "wishstore_db",
	}
	configPool := pgx.ConnPoolConfig{
		ConnConfig: config,
	}
	dbPool, err := pgx.NewConnPool(configPool)
	if err != nil {
		return nil, fmt.Errorf("db error: %v", err)
	}
	db := DB{ConnnectionPool: dbPool}
	return &db, nil
}

```

# api handler
```go
package api

import (
	"fmt"
	"net/http"

	"github.com/KidPudel/learn-go-api/db"
)

type WishesHandler struct {
}

func (handler WishesHandler) ServeHTTP(resWriter http.ResponseWriter, req *http.Request) {
	db, err := db.ConnectDB()
	if err != nil {
		fmt.Println("error while getting db")
		return
	}
	fmt.Println("db connected")
	switch {
	case req.Method == "GET" && len(req.URL.Query()) == 0:
		query := `select name, rate from wishes;`
		// we can utilise models here 
		var name string
		var rate int
		// write to the pointed values
		db.ConnnectionPool.QueryRow(query).Scan(&name, &rate)
		fmt.Fprintf(resWriter, "name: %v, rate: %v", name, rate)
	default:
		fmt.Fprintln(resWriter, "unhandled request")
	}
}

```

In real life, we need automated [[routing]] and [[models]]
