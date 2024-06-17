Index is a **data structure** in a database to process data faster, it works like a indexes at the back of the book, by maintaining  a sorted representation of the data in one or more columns to then easily process and find rows, without having to traverse through all rows. Basically points where the data is

Index is a copy of selected column from a table.


Each row/record has a unique *key* associated to it which distinguishes it from others and *those keys are stored in indexes*.

> Note: each time row with unique key is added, index is automatically updated

When we need to look data by data that is not stored as a key (primary key), we need to ***create our own index***.

```sql
create index saved_words_by_user_word_id
on saved_words (user_id, word_id)
```

This will speed up the query since it has mapping (in this case compound index, can be used in combination in queries), and now, when joining and filtering instead of for each row on looked up table, scan entire right table (saved_words) each time, database can efficiently lookup rows


> NOTE: you also can add some rules on index, like `UNIQUE`


To delete/drop an index
```sql
drop index saved_words_by_user_word_id
on saved_words;
```


# [[types of indexes]]


