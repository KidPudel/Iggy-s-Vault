# Database Function

A named, reusable block of SQL or procedural code stored in the database that accepts parameters and returns a value or result set.

## What it does

Functions can return a scalar value, a row, or a set of rows (`RETURNS TABLE`). They are invoked inside queries (`SELECT`, `INSERT`, `UPDATE`) or via `EXECUTE FUNCTION` in a trigger.

The function body is delimited with `$$` and uses `BEGIN`/`END` for procedural logic (PL/pgSQL).

Volatility categories:
- `IMMUTABLE` — same inputs always give same output; no side effects
- `STABLE` — consistent within a single query; may depend on query context
- `VOLATILE` (default) — may produce different results or have side effects

## Code

```sql
CREATE OR REPLACE FUNCTION add_tax(price NUMERIC)
RETURNS NUMERIC AS $$
BEGIN
  RETURN price * 1.2;
END;
$$ LANGUAGE plpgsql;

SELECT add_tax(100); -- 120
```

## Sources

- https://www.postgresql.org/docs/current/sql-createfunction.html

## Related

- [[trigger function]]
- [[trigger]]
- [[transactions]]

## Process

- What is the difference between a function and a stored procedure in PostgreSQL?
- How does the `IMMUTABLE` volatility category affect query planning?
- What happens when a function declared `IMMUTABLE` has side effects?
