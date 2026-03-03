# HTTP Headers

Key-value pairs sent alongside HTTP requests and responses that carry metadata about the message.

## What it does

Headers are sent before the body. Keys are case-insensitive. Values follow the key after a colon.

Common request headers: `Content-Type`, `Authorization`, `Accept`, `Cache-Control`, `Cookie`
Common response headers: `Set-Cookie`, `Content-Length`, `Location`, `WWW-Authenticate`

`Content-Type` specifies the media type of the body (e.g. `application/json`).
`Content-Length` gives the body size in bytes.
`Authorization` carries credentials (e.g. `Bearer <token>`).
`Cache-Control` directs caching behavior for proxies and browsers.

Custom headers conventionally use the `X-` prefix (deprecated in RFC 6648 but still common).

## Sources

- https://www.rfc-editor.org/rfc/rfc9110

## Related

- [[HTTP]]
- [[Authorization header]]
- [[cookies]]
- [[Payload]]
- [[metadata]]

## Process

- What is the difference between `Content-Type` on a request vs on a response?
- If `Content-Length` is wrong, what happens at the receiving end?
- How do headers interact with CORS preflight requests?
- Where does header injection become a security concern?
