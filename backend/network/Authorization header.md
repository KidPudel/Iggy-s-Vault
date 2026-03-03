# Authorization Header

An HTTP request header that carries credentials or tokens for authenticating a request.

## What it does

Format: `Authorization: <scheme> <credentials>`

Common schemes:
- `Basic` — Base64-encoded `username:password`; insecure without HTTPS
- `Bearer` — an opaque or signed token (e.g. JWT or OAuth access token)
- `Digest` — a hashed version of credentials
- `ApiKey` — a pre-shared key (non-standard, varies by service)

The server reads the header, validates the credentials according to the scheme, and either proceeds or returns `401 Unauthorized`.

## Code

```http
Authorization: Bearer eyJhbGciOiJIUzI1NiJ9...
Authorization: Basic dXNlcjpwYXNz
```

## Sources

- https://www.rfc-editor.org/rfc/rfc7235

## Related

- [[headers]]
- [[authentication]]
- [[JWT]]
- [[base64-encoding]]
- [[hashing]]

## Process

- What is transmitted in plaintext when using Basic auth over HTTP?
- How does a Bearer token differ from embedding credentials directly in the header?
- What does the server return when Authorization is missing vs when it's invalid?
