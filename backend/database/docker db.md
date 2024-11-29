```
migrate -path ./migrations -database "postgres://postgres:postgres@localhost:5432/postgres?sslmode=disable" up
```

```
POSTGRES_ADDRESS=postgres://postgres:postgres@localhost:5432/postgres?sslmode=disable; \
  migrate -path migrations/ -database "$POSTGRES_ADDRESS" -verbose up  
```

```
docker exec -it receipt-analysis-service-db-1 psql -U postgres -d postgres
```
