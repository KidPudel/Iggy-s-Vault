# Repeatable Migrations (Flyway)

Flyway migration scripts that are re-applied every time their content changes, rather than being applied once in version order.

## What it does

Repeatable migrations are named with the `R__` prefix (e.g. `R__refresh_views.sql`). Flyway tracks a checksum of the file. If the checksum changes on the next `migrate` run, the script is re-executed.

They have no version number and always run after all versioned migrations.

Typical uses: views, stored functions, stored procedures, configuration data — objects whose definition changes over time and should always reflect the latest script.

## Sources

- https://documentation.red-gate.com/flyway/flyway-concepts/migrations/repeatable-migrations

## Related

- [[flyway]]
- [[migrations]]
- [[views]]
- [[function]]

## Process

- What does Flyway do when the checksum of a repeatable migration changes?
- What is the difference between a repeatable migration and a versioned migration in terms of idempotency?
- Why would you prefer a repeatable migration for a view over a versioned migration?
