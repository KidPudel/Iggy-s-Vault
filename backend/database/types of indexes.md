
# B-tree
---
default one used in `CREATE INDEX`
Can handle equality and range queries on data that can be stored into some ordering
```
< <= = >= >
```
B-tree can have greater than one keys in nodes and greater than 2 children


# Unique
---
Ensures uniqueness of values in the indexed column
```sql
CREATE UNIQUE INDEX idx_name ON users(name);
```


# Hash
---
Hash indexes store a 32-bit hash code ([[hash function]]) derived from the value of the indexed column
**So this indexes can only handle simple equality comparisons**
```
=
```
# Non-clustered
---
index is **pointing to the data**, while not ordering it
Example: Book with indexes at the back

We can have multiple non-clustered indexes, because they are just references that pointing to the data

# Clustered
---
index is **re-organizes the data** for efficient use
Example: Phone book

We can have only one, because it organizes the data


# Composite
---
Index on 2 or more columns
Real DB Example: We search in many-to-many [[database relationship]], and you want to find saved *word* by *user*




# GiST index (Generalized Search Tree index)
Used for full-text search

```
<<   &<   &>   >>   <<|   &<|   |&>   |>>   @>   <@   ~=   &&
```

# SP-GiST (Space-Partitioned GiST)
```
<<   >>   ~=   <@   <<|   |>>
```
# GIN (Generalized Inverted Index)
```
<@   @>   =   &&
```
## BRIN (Block Range index)
```
<   <=   =   >=   >
```