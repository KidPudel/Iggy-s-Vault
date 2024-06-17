`UNION` is used to combine 2 or more `SELECT` statements, to get all that info in one table result
```sql
select name from users
union
select name from customers
order by name;
```