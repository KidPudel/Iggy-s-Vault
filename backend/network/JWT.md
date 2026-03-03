# JSON Web Token

A compact, self-contained token format that encodes claims as a signed JSON object, used for stateless authentication.

## What it does

A JWT consists of three Base64URL-encoded parts separated by dots: header, payload, signature.

The header specifies the algorithm and token type. The payload contains claims (key-value pairs like `sub`, `exp`, `iat`). The signature is computed over header + payload using a secret or private key.

The server verifies a JWT by recomputing the signature and comparing it — no session lookup needed.

Tokens expire based on the `exp` claim.

## Code

```json
// Header
{ "alg": "HS256", "typ": "JWT" }

// Payload
{ "sub": "user_id", "exp": 1700000000 }

// Signature
HMACSHA256(base64(header) + "." + base64(payload), secret)
```

```
eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIxMjMifQ.signature
```

## Sources

- https://www.rfc-editor.org/rfc/rfc7519

## Related

- [[base64-encoding]]
- [[claims]]
- [[authentication]]
- [[cookies]]
- [[hashing]]

## Process

- What breaks if the secret key leaks?
- How does token expiry interact with a stateless server that never stores tokens?
- What information in the payload can a client read without the secret?
- How does JWT verification differ from session-based lookup at the database level?
