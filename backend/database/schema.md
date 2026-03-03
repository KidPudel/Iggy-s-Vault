# Database Schema

The structure definition of a database: tables, columns, data types, constraints, and relationships.

## What it does

A schema defines what objects exist in a database and how they are shaped. It specifies:
- Table names and column names
- Data types for each column
- Constraints (primary key, foreign key, NOT NULL, UNIQUE, CHECK)
- Relationships between tables

In PostgreSQL and Oracle, "schema" also refers to a named namespace that groups database objects (tables, views, functions). Multiple schemas can coexist in one database, each with its own permission set.

In MySQL, "schema" and "database" are synonymous.

## Sources

- https://www.postgresql.org/docs/current/ddl-schemas.html

## Related

- [[RDBMS]]
- [[migrations]]
- [[foreign key]]
- [[primary key]]
- [[constraints]]

## Process

- What is the difference between a schema change and a data change in the context of migrations?
- How does PostgreSQL use schemas differently from MySQL?
- What does a schema-less database give up compared to a schema-defined one?
