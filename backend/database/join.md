`JOIN` is used to combine 2 or more tables to filter and put into one table desired columns


There is:
 - `left join` (default): to depend on left table
   The process: since we need to get all rows from left table (meaning left is *main character* here), we take each row from it (go row by row) and processing entire right table against it,
 - `right join`: depend only on right table
   The process: reverse from left join
 - `inner join`: depend on similarities of both
 - `outer join`: depend on both independently (grab all)

```sql
select
	sw.user_id,
	w.chinese,
	w.english,
	w.russian,
	w.level
from words w
inner join saved_words sw on sw.word_id = w.id
where sw.user_id = 3412
```
 