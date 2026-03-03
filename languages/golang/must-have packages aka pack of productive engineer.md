# Must-Have Go Packages

**Sync**
- `errgroup` — WaitGroup + error propagation + context cancellation for goroutine groups https://pkg.go.dev/golang.org/x/sync/errgroup

**Config**
- `godotenv` — parse `.env` files https://pkg.go.dev/github.com/joho/godotenv

**Logging**
- `zap` — fast structured logger https://pkg.go.dev/go.uber.org/zap
- `zerolog` — zero-allocation JSON logger https://pkg.go.dev/github.com/rs/zerolog

**Database**
- `gorm` — ORM https://pkg.go.dev/gorm.io/gorm

**HTTP / Router**
- `gin` — fast, good middleware ecosystem https://pkg.go.dev/github.com/gin-gonic/gin
- `chi` — lightweight, stdlib-compatible https://pkg.go.dev/github.com/go-chi/chi/v5
- `echo` — high performance https://pkg.go.dev/github.com/labstack/echo/v4

**Serialization**
- `protobuf` — Protocol Buffers https://pkg.go.dev/google.golang.org/protobuf
- `easyjson` — code-generated fast JSON https://pkg.go.dev/github.com/mailru/easyjson

**Testing**
- `gomock` — generates mock implementations from interfaces https://pkg.go.dev/go.uber.org/mock/gomock

**Distributed**
- `dtm` — distributed transactions (saga, TCC, 2PC) https://pkg.go.dev/github.com/dtm-labs/dtm
