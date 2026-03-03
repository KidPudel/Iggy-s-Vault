# Authentication

The process of verifying that a requester is who they claim to be.

## What it does

Authentication establishes identity by validating a credential against a known record.

Common mechanisms:
- **Password** — hash of submitted password compared against stored hash
- **Token** — server-issued opaque or signed token (e.g. JWT) verified on each request
- **API key** — shared secret sent in headers or query params
- **Certificate** — client presents a TLS certificate signed by a trusted CA
- **OAuth 2.0 / OIDC** — delegates authentication to a third-party identity provider

Multi-factor authentication chains two or more mechanisms.

## Sources

- https://www.rfc-editor.org/rfc/rfc7235

## Related

- [[authorization]]
- [[JWT]]
- [[hashing]]
- [[SSL]]
- [[cookies]]

## Process

- What is the difference between authenticating a user and authenticating a machine/service?
- How does stateless token authentication differ from session-based authentication at the server level?
- What breaks if authentication succeeds but authorization is skipped?
- Where does OAuth 2.0 sit — does it authenticate or authorize?
