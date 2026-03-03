# HTTP Cookie

A small piece of data the server sends to the browser, which the browser stores and automatically resends on subsequent requests to the same domain.

## What it does

Cookies are set via the `Set-Cookie` response header. The browser attaches matching cookies in the `Cookie` request header on every subsequent request.

Key attributes:
- `HttpOnly` — inaccessible to JavaScript; only sent in HTTP requests
- `Secure` — only sent over HTTPS
- `SameSite=Strict|Lax|None` — controls whether the cookie is sent on cross-site requests
- `Expires` / `Max-Age` — sets persistence; without these the cookie is session-scoped (deleted when browser closes)
- `Domain` / `Path` — scopes which requests include the cookie

## Code

```http
Set-Cookie: session=abc123; HttpOnly; Secure; SameSite=Strict; Max-Age=3600
```

## Sources

- https://www.rfc-editor.org/rfc/rfc6265

## Related

- [[JWT]]
- [[authentication]]
- [[headers]]
- [[CORS]]

## Process

- What does `HttpOnly` prevent, and what attack does that mitigate?
- If a cookie has `SameSite=None`, what additional attribute is required and why?
- How do cookies differ from localStorage in terms of what the server can read?
- When a cookie has no `Expires` or `Max-Age`, when exactly is it deleted?
