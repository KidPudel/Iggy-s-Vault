# Database Cursor

A server-side pointer that tracks position in a query result set and fetches rows one at a time or in batches.

## What it does

1. A query executes and the result set is held on the database server
2. The cursor is opened, pointing to the first row
3. `FETCH` retrieves the current row (or a batch) and advances the pointer
4. The cursor tracks the current position and knows when the result set is exhausted
5. The cursor is closed to release server resources

Cursors are useful when the full result set is too large to load into memory at once.

## Code

```sql
DECLARE my_cursor CURSOR FOR SELECT id, name FROM users;
OPEN my_cursor;
FETCH NEXT FROM my_cursor;
CLOSE my_cursor;
DEALLOCATE my_cursor;
```

## Sources

- https://www.postgresql.org/docs/current/plpgsql-cursors.html

## Related

- [[transactions]]
- [[database basics]]
- [[connection pool]]

## Process

- What happens to a cursor if the transaction it belongs to is rolled back?
- How does a cursor differ from loading the entire result set into application memory?
- What resource does an open cursor hold on the server, and why does failing to close it matter?
