
# Basic terminology
- `Database`: A collection of relations
- `Table`/`Relation`: (consists of 2 parts) A collection of data that is described by columns (itself contains itself, for example `wish_lists` table, contains wish lists)
  - `Schema`: description (or metadata), **blueprint**, how to structure a table 
  - `Instance`: An actual set of data satisfying the schema, for example: 1st `Rollerblades`, 2nd `Chair`
- `Field` (Column, Attribute): Part of defining schema
- `Record` (Row, Table): One instance of a data that is follows a whole schema of table


> **_We design database based on logic/semantics that we want to convey._**

### Note about how to structure a database
tables **_must have only primitive types_** like: `strings`, `numerics`, `data and time`

**`For examples`**: We have a **wishlists** (_a collection of **wisheses**_), each wishlists is themed ("holydays", "birthday", etc.),
but since we cannot have in DB just a table `wishlists`, that has field `wisheses` (since it is not a primitive and a bad practice), we need to divide to a simplest form, so at the base, those entities are:
- `wishes`: table, with wishes description schema
- `wishlists`: table, with name for the wishes's group

**`The way to go`**: Associate a relation, to where it belongs (groups, concentrate) in a DB (based on semantics).  
In this example: wishes is associated with it's list (wishlists), so by associating name of the wishes group to each wishes, we end up with wisheses, that at its own instances all bounded to group/groups.


# Basic syntax
## Data definition

### creating

```sql
CREATE DATABASE wishesstore;
```

```sql
CREATE TABLE wish_lists (
  id serial primary key,
  talk_id serial
  name varchar(50)
  constraint fk_talks
	  foreign key (talk_id)
		  references talks(id)
);
```

```sql
CREATE TABLE reference_receipts (
    id UUID NOT NULL UNIQUE,
    type INT NOT NULL,
    CONSTRAINT fk_receipt
        FOREIGN KEY(id)
            REFERENCES receipts(id)
            ON DELETE CASCADE
);
```


> **NOTE**: Your user may need a [[db role]] assigned to them to create a DB
### altering

```sql
ALTER TABLE wish_lists ADD COLUMN importance int;
```

```sql
ALTER TABLE wish_lists ALTER COLUMN importance TYPE float;
```

```sql
ALTER TABLE wish_lists DROP COLUMN importance;
```
additionally could be specified `CASCADE` at the end, to also delete all referenced places



---
## Data manipulation
```sql
INSERT INTO wish_lists(name) VALUES ('test list');
```

```sql
UPDATE wishes SET rate = 0.0 WHERE name LIKE '%game%' OR name LIKE "%oil" OR name LIKE "virtual%";
```

`LIKE` lets you to match values
`LIKE` operator uses two signs:
- `%` match any sequence of values with length of 0 or more
- `_` match any single character
>**NOTE**: Also available `NOT LIKE` for reverse and `ILIKE` for case-insensitive

Alternative:

| **Operator Name** | **Substitute with Description** |
| ----------------- | ------------------------------- |
| ~~                | It is used instead of like      |
| ~~*               | It is used instead of ilike     |
| !~~               | It is used instead of not like  |
| !~~*              | It is used instead of not ilike |

```sql
DELETE FROM wishes WHERE rate < 8.0;
```


---

## queries

```sql
SELECT * FROM wishes WHERE rate = 10;
```

```sql
SELECT wl.name, w.name, w.rate 
FROM wisheses w
INNER JOIN wish_lists

WHERE wl.name LIKE '%birthday%' 
GROUP BY w.rate;
```


# [[sql data **types**]]


# conversion from json
```sql
where (payload::json)->>'externalID' = '9b733e35-1169-4247-a213-af811759edba'
```

#database

# Aggregate functions
function that takes set of values and return singe value.
- `MIN()`
- `MAX()`
- `COUNT()`
- `SUM()`
- `AVG()`

# HAVING
---
`HAVING` was added because `WHERE` keyword cannot be used with aggregate functions
```sql
select count(usr_id), country
from users
where country != "Finland"
group by country
having count(usr_id) > 5;
```


# Views


# Procedures


# Explain


# GROUP BY
---
Grouping rows that have the same value into a summary rows, like "find the number of customers in **each country**"
```sql
select count(id), country
from customers
group by country;
```
It is of often used with aggregate functions to group the result-set by one or more columns 
```sql
select count(usr_id), country
from users
where country != "Finland"
group by country
having count(usr_id) > 5;
```

Or another example: list users and their purchases that **made purchases greater than 5000 dollars**
User and purchase has one to many relationship, so users will appear in the table, so we can should join here
so here we need to group by users and find the sums
```sql
select u.id, sum(p.price)
from users u
join purchases p on u.id = u.user_id
group by u.id
having sum(p.price) > 5000

```

# DISTINCT
unique rows