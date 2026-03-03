# Data Dimension

A categorical attribute in a data warehouse that provides context for measuring numeric facts.

## What it does

Dimensions are the entry points by which facts are filtered, grouped, or aggregated. Common dimensions: customer, product, store, time.

In an OLAP cube, each axis represents a dimension. A fact (e.g. "units sold") sits at the intersection of dimension values (e.g. product=Widget, store=LA, month=January).

Dimension tables in a star schema contain descriptive attributes (name, region, category) and a primary key that foreign keys in the fact table reference.

## Sources

- [TODO: find source]

## Related

- [[OLAP]]
- [[OLAP cube]]
- [[data fact]]
- [[multi-dimensional data]]

## Process

- What is the relationship between a dimension and the fact table in a star schema?
- How does adding a new dimension change the shape of the OLAP cube?
- What is the difference between a dimension value and a fact value?
