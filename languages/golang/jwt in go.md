[[JWT]] in go can be accomplished with [this package](https://golang-jwt.github.io/jwt/usage/create/)


# encoding
symmetric is meant for peer-to-peer and share private keys
asymmetric signed by sender with private key, and checked by receiver by a public key

# creation
```go
token := jwt.NewWithClaims(jwt.SigningMethodES256,
jwt.MapClaims{
	"user_id": "****",
	"live_for": "100000"
}
)
signedToken, err := token.SignedString("key")
```


# parsing
## keyfunc
Keyfunc used by the Parse methods is the callback for supplying key for verification to the inner process of verification that token has not been temperated.


```go
token, err := jwt.Parse("jwtToken", func(token *jwt.Token) (interface{}, error) {
	return []byte("key"), nil
})
```