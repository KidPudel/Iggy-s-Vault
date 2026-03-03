4 key properties that define [[transactions]]

# Atomicity
Each statement in transaction is treated as a **single unit**. Either the entire statement is executed or none.
This prevents data loss and corruption

```sql
BEGIN; 
UPDATE accounts SET balance = balance + 100.00 WHERE acctnum = 12345; 
UPDATE accounts SET balance = balance - 100.00 WHERE acctnum = 789; 
COMMIT;
```
or both will happen or none

# Consistency
Transactions only make changes to tables in predefined, predictable ways. Prevents unintended consequences

# Isolation
**When multiple users are reading and writing** from the same table, isolation of their transactions ensures that the concurrent transactions don't interfere with or affect one another. **Each request can occur as though they were occurring one by one, even though they're actually occurring simultaneously**.
There is [[4 levels of isolation]]


# Durability
Ensures that changes to you data made by successfully executed transactions will be saved **even in the event of system failure**
saves data to other to [[WAL]] file
