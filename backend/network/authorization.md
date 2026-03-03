# Authorization

The process of determining what an authenticated identity is permitted to do.

## What it does

Authorization runs after authentication and checks permissions against a policy.

Common models:
- **RBAC** (Role-Based Access Control) — permissions attached to roles, roles attached to users
- **ABAC** (Attribute-Based) — policy rules evaluate attributes of the subject, resource, and environment
- **ACL** (Access Control List) — explicit list of who can access each resource
- **Scope-based** (OAuth 2.0) — tokens carry declared scopes; server checks token scope against required scope

Authorization can be enforced at the API gateway, middleware, or service layer.

## Sources

- https://www.rfc-editor.org/rfc/rfc6749

## Related

- [[authentication]]
- [[JWT]]
- [[claims]]
- [[middleware]]

## Process

- What is the difference between a permission and a role?
- If authorization is checked only at the API gateway, what happens when services call each other internally?
- How do JWT claims relate to authorization decisions?
- What breaks when roles are too coarse-grained?
