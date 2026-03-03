# Go Backend — Lookup Reference

> This is a dictionary, not a textbook. Come here ONLY after you tried to recall something and failed.
> Do NOT read this cover to cover. That's the illusion of competence.
>
> - To learn: [[Go Backend Deep Learning Method]]
> - To test yourself: [[Go Backend Interview Questions Bank]]

---

# Go Language

## Types and Declarations
```go
// Value types
var x int = 42
var f float64 = 3.14
var s string = "hello"
var b bool = true
var arr [3]int = [3]int{1, 2, 3}

// Reference types
var sl []int = []int{1, 2, 3}
var m map[string]int = map[string]int{"a": 1}
var ch chan int = make(chan int)

// Short declaration
x := 42

// Zero values
// int: 0, float: 0.0, string: "", bool: false
// pointer/slice/map/channel/interface/function: nil
```

## Structs
```go
type User struct {
    ID        int64     `json:"id" db:"id"`
    Name      string    `json:"name" db:"name"`
    CreatedAt time.Time `json:"created_at" db:"created_at"`
}

func (u *User) SetName(name string) { u.Name = name } // pointer receiver
func (u User) FullName() string     { return u.Name }  // value receiver

type Admin struct {
    User // embedding
    Permissions []string
}
```

## Interfaces
```go
type Reader interface {
    Read(p []byte) (n int, err error)
}

type ReadWriter interface {
    Reader
    Writer
}

func Print(v any) { fmt.Println(v) }

// Type assertion
val, ok := i.(string)

// Type switch
switch v := i.(type) {
case string:
case int:
}
```

## Generics
```go
func Map[T any, R any](slice []T, fn func(T) R) []R {
    result := make([]R, len(slice))
    for i, v := range slice {
        result[i] = fn(v)
    }
    return result
}

type Number interface {
    ~int | ~int64 | ~float64
}
```

## Errors
```go
// Wrapping
return fmt.Errorf("operation failed: %w", err)

// Custom error type
type NotFoundError struct {
    Resource string
    ID       int64
}
func (e *NotFoundError) Error() string {
    return fmt.Sprintf("%s with id %d not found", e.Resource, e.ID)
}

// Checking
errors.Is(err, sql.ErrNoRows)
var nfe *NotFoundError
errors.As(err, &nfe)

// Sentinel
var ErrNotFound = errors.New("not found")
```

## Defer, Panic, Recover
```go
defer f.Close() // LIFO execution on function return

// Recover from panic
defer func() {
    if r := recover(); r != nil {
        log.Printf("recovered: %v", r)
    }
}()
```

---

# Concurrency

## Channels
```go
ch := make(chan int)     // unbuffered
ch := make(chan int, 10) // buffered

func producer(out chan<- int) { out <- 42 }  // send-only
func consumer(in <-chan int)  { v := <-in }  // receive-only

for v := range ch { process(v) }

select {
case v := <-ch1:
case ch2 <- value:
case <-ctx.Done():
    return ctx.Err()
default:
}
```

## Channel Axioms
| Operation          | nil channel    | closed channel | open channel       |
| ------------------ | -------------- | -------------- | ------------------ |
| Send `ch <- v`     | blocks forever | **panic**      | sends or blocks    |
| Receive `<-ch`     | blocks forever | zero value     | receives or blocks |
| Close `close(ch)`  | **panic**      | **panic**      | closes             |

## Sync Primitives
```go
// WaitGroup
var wg sync.WaitGroup
wg.Add(1)
go func() { defer wg.Done(); work() }()
wg.Wait()

// Mutex
var mu sync.Mutex
mu.Lock()
defer mu.Unlock()

// RWMutex
var rw sync.RWMutex
rw.RLock() / rw.RUnlock()  // many readers
rw.Lock()  / rw.Unlock()   // one writer

// Once
var once sync.Once
once.Do(func() { init() })

// sync.Map
var m sync.Map
m.Store("key", "value")
v, ok := m.Load("key")
```

## Patterns
```go
// Pipeline
func gen(nums ...int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for _, n := range nums { out <- n }
    }()
    return out
}

// Worker Pool
func workerPool(jobs <-chan Job, results chan<- Result, n int) {
    var wg sync.WaitGroup
    for i := 0; i < n; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            for job := range jobs { results <- process(job) }
        }()
    }
    go func() { wg.Wait(); close(results) }()
}

// Context Cancellation
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()

// Errgroup
g, ctx := errgroup.WithContext(ctx)
g.Go(func() error { return fetchUsers(ctx) })
if err := g.Wait(); err != nil { ... }
```

## Runtime

**GMP**: G = Goroutine, M = OS Thread, P = Processor (logical CPU, holds run queue). GOMAXPROCS = num CPUs.

**GC**: Concurrent tri-color mark-and-sweep. White/grey/black. GOGC default 100.

**Escape analysis**: `go build -gcflags="-m"`

**Slice header**: `{ array unsafe.Pointer, len int, cap int }`

**Map**: hash table with buckets (8 KV pairs per bucket). Not concurrent-safe. Random iteration order.

---

# Standard Library

## net/http
```go
mux := http.NewServeMux()
mux.HandleFunc("GET /users/{id}", getUser)
mux.HandleFunc("POST /users", createUser)

srv := &http.Server{
    Addr:         ":8080",
    Handler:      mux,
    ReadTimeout:  5 * time.Second,
    WriteTimeout: 10 * time.Second,
    IdleTimeout:  120 * time.Second,
}
srv.ListenAndServe()

// Handler
func getUser(w http.ResponseWriter, r *http.Request) {
    id := r.PathValue("id")
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(response)
}

// Middleware
func logging(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        start := time.Now()
        next.ServeHTTP(w, r)
        log.Printf("%s %s %v", r.Method, r.URL.Path, time.Since(start))
    })
}
```

## context
```go
ctx := context.Background()
ctx, cancel := context.WithCancel(ctx)
ctx, cancel := context.WithTimeout(ctx, 5*time.Second)
ctx, cancel := context.WithDeadline(ctx, deadline)
ctx = context.WithValue(ctx, key, value)
// Always: defer cancel()
// Always: first parameter
```

## database/sql
```go
db, err := sql.Open("postgres", connString)
db.SetMaxOpenConns(25)
db.SetMaxIdleConns(5)
db.SetConnMaxLifetime(5 * time.Minute)

// Single row
err = db.QueryRowContext(ctx,
    "SELECT id, name FROM users WHERE id = $1", id,
).Scan(&user.ID, &user.Name)

// Multiple rows
rows, err := db.QueryContext(ctx, "SELECT id, name FROM users WHERE active = $1", true)
defer rows.Close()
for rows.Next() {
    rows.Scan(&u.ID, &u.Name)
}
rows.Err()

// Transaction
tx, err := db.BeginTx(ctx, nil)
defer tx.Rollback()
tx.ExecContext(ctx, "UPDATE ...", args...)
tx.Commit()
```

## encoding/json
```go
data, err := json.Marshal(user)
err := json.Unmarshal(data, &user)
json.NewEncoder(w).Encode(response)
json.NewDecoder(r.Body).Decode(&request)

type User struct {
    ID    int64  `json:"id"`
    Name  string `json:"name"`
    Email string `json:"email,omitempty"`
}
```

## testing
```go
func TestAdd(t *testing.T) {
    tests := []struct {
        name     string
        a, b     int
        expected int
    }{
        {"positive", 1, 2, 3},
        {"zero", 0, 0, 0},
    }
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            if got := Add(tt.a, tt.b); got != tt.expected {
                t.Errorf("Add(%d, %d) = %d, want %d", tt.a, tt.b, got, tt.expected)
            }
        })
    }
}

// go test ./...
// go test -race ./...
// go test -cover ./...
// go test -bench=. ./...
```

---

# Common Libraries

| Category   | Libraries                              |
| ---------- | -------------------------------------- |
| Router     | chi, gin, echo, stdlib (1.22+)         |
| SQL        | sqlx, pgx, GORM, sqlc                 |
| Migrations | goose, golang-migrate                  |
| Config     | viper, envconfig, koanf               |
| Logging    | slog (1.21+), zerolog, zap            |
| Validation | go-playground/validator               |
| Testing    | testify, gomock, testcontainers-go    |
| gRPC       | google.golang.org/grpc, buf           |
| Kafka      | segmentio/kafka-go, confluent-kafka-go |
| Redis      | go-redis/redis                        |
| Auth       | golang-jwt/jwt                        |
| DI         | wire, fx, dig                         |

---

# SQL

## Syntax
```sql
-- Joins
SELECT u.name, o.total
FROM users u
INNER JOIN orders o ON u.id = o.user_id;  -- only matching
LEFT JOIN  -- all from left + matching right
RIGHT JOIN -- all from right + matching left
FULL OUTER JOIN -- all from both
CROSS JOIN -- cartesian product

-- Aggregation
SELECT department, COUNT(*), AVG(salary)
FROM employees
GROUP BY department
HAVING AVG(salary) > 50000;

-- Window functions
SELECT name, salary,
       RANK() OVER (PARTITION BY department ORDER BY salary DESC) as rank
FROM employees;

-- CTE
WITH active AS (
    SELECT id, name FROM users WHERE status = 'active'
)
SELECT * FROM active;

-- Other
SELECT COALESCE(nickname, name, 'anon') FROM users;
SELECT DISTINCT department FROM employees;
UNION / UNION ALL
EXISTS (SELECT 1 FROM ...)
EXPLAIN ANALYZE SELECT ...;

CASE WHEN salary > 100000 THEN 'high'
     WHEN salary > 50000 THEN 'mid'
     ELSE 'low' END
```

## Indexes
```sql
-- B-tree (default): equality + range
-- Hash: equality only
-- GIN: full-text, JSONB, arrays
-- GiST: geometric, spatial
-- BRIN: large naturally ordered tables

CREATE INDEX idx_name ON users (status, created_at);  -- composite (leftmost prefix rule)
CREATE UNIQUE INDEX idx_email ON users (email);
CREATE INDEX idx_active ON users (email) WHERE status = 'active';  -- partial
CREATE INDEX idx_lower ON users (LOWER(email));  -- expression
```

## Isolation Levels
| Level            | Dirty Read | Non-Repeatable | Phantom |
| ---------------- | ---------- | -------------- | ------- |
| Read Uncommitted | yes        | yes            | yes     |
| Read Committed   | no         | yes            | yes     |
| Repeatable Read  | no         | no             | yes     |
| Serializable     | no         | no             | no      |

PostgreSQL default: Read Committed.

## Locking
```sql
SELECT * FROM jobs WHERE status = 'pending' LIMIT 1 FOR UPDATE;
SELECT * FROM jobs WHERE status = 'pending' LIMIT 1 FOR UPDATE SKIP LOCKED;
```

---

# System Design Building Blocks

**Load Balancer**: round-robin, least connections, IP hash. L4 (TCP) vs L7 (HTTP). nginx, HAProxy, ALB/NLB.

**Cache strategies**: cache-aside, write-through, write-behind, read-through. Eviction: LRU, LFU, TTL.

**CAP**: during partition → choose Consistency (CP) or Availability (AP).

**Consistency**: strong, eventual, causal.

**Scaling**: vertical (bigger machine) vs horizontal (more machines). Read replicas. Sharding for writes.

**Rate limiting**: token bucket, sliding window, fixed window, leaky bucket.

**Replication**: primary-replica, sync vs async.

**Partitioning**: range, hash, list (horizontal). Vertical = split columns.

---

# API Design

## REST
```
GET    /api/v1/users          → list
GET    /api/v1/users/{id}     → get
POST   /api/v1/users          → create
PUT    /api/v1/users/{id}     → full update
PATCH  /api/v1/users/{id}     → partial update
DELETE /api/v1/users/{id}     → delete
```

## Status Codes
- `200 OK`, `201 Created`, `204 No Content`
- `400 Bad Request`, `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `409 Conflict`, `422 Unprocessable Entity`, `429 Too Many Requests`
- `500 Internal Server Error`, `502 Bad Gateway`, `503 Service Unavailable`

## Pagination
```
?page=2&per_page=20           -- offset-based
?cursor=abc123&limit=20       -- cursor-based
```

## gRPC
Protocol Buffers over HTTP/2. Unary, server streaming, client streaming, bidirectional. Code generation from .proto files.

---

# Architecture

## Project Structure
```
cmd/
    api/
        main.go
internal/
    domain/
        user.go              ← entities, interfaces
    service/
        user_service.go      ← business logic
    repository/
        postgres/
            user_repo.go     ← implementation
    handler/
        user_handler.go      ← HTTP/gRPC
    middleware/
pkg/
migrations/
docker-compose.yml
Makefile
```

## SOLID
- **S**: one struct/package, one responsibility
- **O**: extend via interfaces, don't modify
- **L**: implementations are interchangeable
- **I**: small interfaces (`io.Reader`, not `io.ReadWriteCloserSeeker`)
- **D**: depend on interfaces, not concrete types

## Patterns
```go
// Dependency Injection
type UserService struct {
    repo UserRepository
}
func NewUserService(repo UserRepository) *UserService {
    return &UserService{repo: repo}
}

// Graceful Shutdown
quit := make(chan os.Signal, 1)
signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
<-quit
ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
defer cancel()
srv.Shutdown(ctx)

// Retry with Backoff
for attempt := 0; attempt < maxRetries; attempt++ {
    if err := doRequest(); err == nil { return nil }
    time.Sleep(time.Duration(math.Pow(2, float64(attempt))) * baseDelay + jitter())
}
```

**Circuit Breaker**: Closed → Open (failing) → Half-Open (testing recovery).

**Outbox**: write event to DB in same transaction, separate process publishes to queue.

---

# Docker

## Dockerfile (multi-stage, Go)
```dockerfile
FROM golang:1.22-alpine AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o /app/server ./cmd/api

FROM alpine:3.19
RUN apk --no-cache add ca-certificates
COPY --from=builder /app/server /server
EXPOSE 8080
CMD ["/server"]
```

## Docker Compose
```yaml
services:
  api:
    build: .
    ports: ["8080:8080"]
    environment:
      - DB_HOST=postgres
      - REDIS_HOST=redis
    depends_on:
      postgres:
        condition: service_healthy
  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: myapp
      POSTGRES_PASSWORD: secret
    volumes: [pgdata:/var/lib/postgresql/data]
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
  redis:
    image: redis:7-alpine
volumes:
  pgdata:
```

## Commands
```bash
docker build -t myapp:latest .
docker run -d -p 8080:8080 --name myapp myapp:latest
docker ps / docker logs myapp / docker exec -it myapp sh
docker system prune -a
```

---

# Observability

- **Logging**: structured (JSON), slog/zerolog/zap, correlation IDs
- **Metrics**: RED (Rate, Errors, Duration), Prometheus + Grafana
- **Tracing**: distributed spans, Jaeger, OpenTelemetry
- **Alerting**: SLO-based, not per-error

# CI/CD

checkout → lint (golangci-lint) → test → build → push image → deploy

Blue-green / canary deployment. Database migrations before app deploy.

# Auth

- **JWT**: header.payload.signature, access (short) + refresh (long), stateless
- **OAuth 2.0**: Authorization Code (web), Client Credentials (service-to-service)
- **API Keys**: simple service-to-service
- **Sessions**: server-side state, cookie-based
