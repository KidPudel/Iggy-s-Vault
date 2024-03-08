To implement a getter of the existing instance we can do it by declaring private variable in a global scope of a package, and work with it.

[thank you](https://blog.matthiasbruns.com/golang-singleton-pattern)
# example
## without singleton
```go
package db

import (
	"fmt"

	"github.com/jackc/pgx"
)

type DB struct {
	ConnnectionPool *pgx.ConnPool
}

func (db *DB) Close() {
	defer db.ConnnectionPool.Close()
}

func ConnectDB() (*DB, error) {
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
## with singleton
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