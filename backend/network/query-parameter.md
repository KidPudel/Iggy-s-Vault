# Query Parameter

An optional key-value pair appended to a URL after `?` used to pass additional data or filter a resource.

## What it does

Query parameters appear after `?` in a URL, formatted as `key=value`. Multiple parameters are separated by `&`. They are used for filtering, sorting, pagination, and search — not for identifying a specific resource.

## Code

```
GET /search?q=keyword&sortBy=date&page=2

# Multiple parameters:
?filter=active&limit=20&offset=40
```

## Sources

- https://restfulapi.net/resource-naming/

## Related

- [[path-parameter]]
- [[REST]]
- [[headers]]

## Process

- What is the structural difference between a query parameter and a path parameter in a URL?
- How does a server distinguish between a parameter used for filtering vs one used for resource identification?
- What encoding is required for special characters within query parameter values?
