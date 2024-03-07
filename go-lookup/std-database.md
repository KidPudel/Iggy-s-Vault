

# Open database
`Open` opens a database specified by its:
1. driver name
2. data-specific data, like database name and connection information

> **NOTE 1**: `Open` may just validate its arguments without a connection to a database. To verify that the data source name is valid, call `DB.Ping`

> **NOTE 2**: Most users will open database via driver specific implementations of connection handlers 


## drivers
Go's standard library provides package [[database-driver]] `database/sql/driver` that defines interface to be implemented by database drivers as used (to be used) by package `sql`


[List of already implemented SQL drivers](https://go.dev/wiki/SQLDrivers)



# getting result from query

When executing query, we can extract the results into the values ***pointed*** (because we want to get the results in it), with `Scan`

```go
query := `select name, rate from wishes;`
		var name string
		var rate int
		// write to the pointed values
		db.ConnnectionPool.QueryRow(query).Scan(&name, &rate)
		fmt.Fprintf(resWriter, "name: %v, rate: %v", name, rate)
```