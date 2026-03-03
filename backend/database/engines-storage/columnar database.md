# Columnar Database

A database storage format that organizes data by column rather than by row.

## What it does

In a columnar layout, all values for a single column are stored contiguously on disk. A row identifier links values across columns.

Aggregate queries (SUM, AVG, COUNT) only read the relevant column(s), skipping unneeded data. This makes read-heavy analytical workloads fast.

Ordering data within columns enables further compression and range scan acceleration.

Bulk inserts are fast (appending to column files). Single-row inserts in the middle of the dataset and updates are slow because they require rewriting column segments.

## Sources

- https://clickhouse.com/docs/en/intro
- https://cassandra.apache.org/doc/latest/

## Related

- [[OLAP]]
- [[DBMS]]
- [[partitioning]]
- [[ClickHouse]]

## Process

- Why is a columnar layout faster for SUM(revenue) than a row-based layout?
- What makes single-row updates expensive in a columnar store?
- Where does a columnar database fit relative to OLTP vs OLAP workloads?
