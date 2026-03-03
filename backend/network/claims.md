# JWT Claims

Key-value pairs in a JWT payload that carry statements about the subject or token metadata.

## What it does

Three categories:
- **Registered** — predefined, interoperable names from RFC 7519: `iss` (issuer), `sub` (subject), `aud` (audience), `exp` (expiration), `iat` (issued at)
- **Public** — custom claims registered in the IANA JWT registry to avoid collisions
- **Private** — custom claims agreed upon between specific parties; not registered anywhere

Claims are Base64URL-encoded but not encrypted — anyone with the token can read them.

## Code

```json
{
  "sub": "user_id_123",
  "iat": 1700000000,
  "exp": 1700003600,
  "admin": true
}
```

## Sources

- https://www.rfc-editor.org/rfc/rfc7519#section-4

## Related

- [[JWT]]
- [[authorization]]
- [[base64-encoding]]
- [[Payload]]

## Process

- Since claims are readable without the secret, what data should never be put in a claim?
- How does `exp` enforcement work on the server side if the server never stores tokens?
- What happens if `aud` is set but the server doesn't validate it?
