# Path Parameter

A variable segment embedded in a URL path used to identify a specific resource.

## What it does

Path parameters are part of the URL path itself, separated by `/`. They define a hierarchical relationship between resources and point to a specific item by identity.

The server extracts the value from the URL segment at a fixed position defined by the route pattern.

## Code

```
GET /users/123
GET /posts/456/comments/789

# Route pattern:
/users/:id  →  id = "123"
```

## Sources

- https://restfulapi.net/resource-naming/

## Related

- [[query-parameter]]
- [[REST]]
- [[headers]]

## Process

- What is the structural difference between a path parameter and a query parameter in a URL?
- How does a path parameter encode a hierarchical relationship between resources?
- What happens when a path parameter value contains a `/` or special character?
