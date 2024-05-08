Constraints enforces data integrity rules and maintain the consistency of the data within a table

When creating *some* of the `constrants` it automatically adds an indexing to the table
- Unique constraints
- Primary key constraints
but some of them don't do that
- Foreign key constraints
- `Check` constraints: conditional checks
- `not null` constraints: conditional checks

```sql
alter table saved_words
add constraint unique_saved_words_for_user
unique (user_id, word_id)
```