# database/sql

Standard library SQL client. `Open` returns a `*DB` with a connection pool. Call `Ping` to verify the connection. Use `Scan` to read query results into variables.

https://pkg.go.dev/database/sql

```go
db.QueryRow("select name from users where id=$1", id).Scan(&name)
```
