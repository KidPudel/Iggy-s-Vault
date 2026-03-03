# OLAP

Online Analytical Processing — a category of database workloads optimized for multi-dimensional queries over large datasets.

## What it does

OLAP systems organize data into dimensions and facts. Queries aggregate facts (sales revenue, unit counts) across dimension combinations (by region, by product, by quarter).

The core data structure is the [[OLAP cube]] — a multi-dimensional representation of data where each axis is a dimension.

OLAP queries typically read many rows but few columns, scan large ranges, and return aggregated results. They are read-heavy and rarely update individual rows.

Columnar storage is the common physical implementation because it enables fast column scans.

## Sources

- [TODO: find source]

## Related

- [[OLAP cube]]
- [[columnar database]]
- [[data dimension]]
- [[data fact]]
- [[ClickHouse]]

## Process

- How does an OLAP query pattern differ from an OLTP query pattern at the storage level?
- What does "drill down" mean in OLAP, and which dimension is being navigated?
- Why is columnar storage a natural fit for OLAP workloads?
