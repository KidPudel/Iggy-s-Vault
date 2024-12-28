Is the database object that gets executed *triggered* when specific event occurs associated with database table, like `SELECT`, `UPDATE`, `INSERT`, or `DELETE` to do something

event on actions in database

syntax:
```sql
create trigger [trigger_name]   
[before | after]    
{insert | update | delete}    
on [table_name]    
[for each row]    
[trigger_body]
```