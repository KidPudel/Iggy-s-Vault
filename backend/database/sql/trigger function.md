# Trigger Function

A special PostgreSQL function with return type `TRIGGER` that executes automatically when a trigger fires.

## What it does

A trigger function operates on implicit `NEW` and `OLD` records:
- `NEW` — the row being inserted or updated
- `OLD` — the previous row state (UPDATE and DELETE only)

It accepts no explicit parameters. It must return either `NEW` (allow and optionally modify the operation), `OLD`, or `NULL` (abort the operation for row-level triggers).

The function is attached to a table via a `CREATE TRIGGER` statement that specifies the event (INSERT, UPDATE, DELETE) and timing (BEFORE, AFTER, INSTEAD OF).

## Code

```sql
CREATE FUNCTION check_version()
RETURNS TRIGGER AS $$
BEGIN
  IF NEW.version != OLD.version + 1 THEN
    RAISE EXCEPTION 'version must increment by 1';
  END IF;
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER enforce_version
BEFORE UPDATE ON my_table
FOR EACH ROW EXECUTE FUNCTION check_version();
```

## Sources

- https://www.postgresql.org/docs/current/trigger-definition.html

## Related

- [[trigger]]
- [[function]]
- [[transactions]]

## Process

- What is the difference between a BEFORE trigger returning NULL vs returning NEW?
- How does a trigger function interact with the transaction that caused the trigger to fire?
- What happens if a trigger function raises an exception?
