# JWT in Go

https://pkg.go.dev/github.com/golang-jwt/jwt/v5

```go
// create
token := jwt.NewWithClaims(jwt.SigningMethodES256, jwt.MapClaims{"user_id": "..."})
signed, _ := token.SignedString(privateKey)

// parse
token, _ := jwt.Parse(signed, func(t *jwt.Token) (any, error) {
	return publicKey, nil
})
```
