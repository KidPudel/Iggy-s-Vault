# Flyway

A database migration tool that versions and applies SQL scripts to a database in a controlled, ordered sequence.

## What it does

Flyway tracks which migrations have been applied in a `flyway_schema_history` table. On each run it compares this table against migration files on disk and applies any pending ones in version order.

Migration file naming:
- Versioned: `V{version}__{description}.sql` — applied once, in order
- Repeatable: `R__{description}.sql` — re-applied whenever their checksum changes

Key commands:
- `migrate` — apply pending migrations
- `validate` — check applied migrations match scripts on disk (checksum comparison)
- `info` — show migration state
- `repair` — remove failed entries from history table; recalculate checksums
- `baseline` — mark an existing database as the starting point

## Code

```shell
flyway -configFiles=./flyway.conf migrate
flyway -url=jdbc:postgresql://host/db -user=postgres -password='' migrate
```

## Sources

- https://documentation.red-gate.com/flyway

## Related

- [[migrations]]
- [[Repeatable Migrations]]
- [[database basics]]
- [[schema]]

## Process

- What does `validate` catch that `migrate` alone would miss?
- What is the difference between a failed migration and a missing migration in Flyway's history table?
- When would you use a repeatable migration instead of a versioned one?
- What does `repair` do vs re-running a migration manually?
