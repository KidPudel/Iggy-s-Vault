> **_Now lets think of a relations between each table_**

# Relationship types
## One to Many / Many to One
1 to many: When one record groups (bounding) depending on it, records (from other table), by appearing in other records multiple times (1 record, appears in many (many depend/relates on it) other records from other table)
`Example`: 
- **One** `student` can have **many** `projects` assigned to it, or **One** `project` assigned to **many** `students` to be done in collaboration
- **One** `wishlist` can store **Many** `wishes`

even if semantics are many to one (many projects, to one student) still, we think of it as one to many (one to many) what is the bigger, more important, has higher weight, A student in this case.

We always reference more important thing (that is **ONE** to many) in many parts  (*it is logically*)

- Person on its own
- Project is taken by someone (person), → therefore, we reference a person in a project**s**
- since we don’t store complex types like lists in a table, we store the thing that can be *common across many things.*

## Many to Many
Many to Many: Both tables could have their parts of records in another table.  
`Example`:
- **One** `employee` could call to **Many** `customers`, _AND_ **One** `customer` could be contacted to **Many** `employees`
- **One** `wishlist` could be viewed by **Many** `users`, _AND_ **One** `user` could view **Many** `wishlists`

## One to One
The most rare type of relation, where only **one** record depend on **one**
`Example`:
- **One** `Employee` could have **One** `Individual valid identity card`, and **One** `identity card` could be associated with **One** `Empoyee`

# Right mindset
even if semantics are many to one (many projects, to one student) still, we think of it as one to many (one to many) what is the bigger, more important, has higher weight, A student in this case.

We always reference more important thing (that is **ONE** to many) in many parts  (*it is logically*)

- Person on its own
- Project is taken by someone (person), → therefore, we reference a person in a project**s**
- since we don’t store complex types like lists in a table, we store the thing that can be *common across many things.*
