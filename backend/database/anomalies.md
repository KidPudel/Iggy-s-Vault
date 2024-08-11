- Dirty read: reading data that **has not been committed** (for example accessing from different session (connection))

|Session A|Session B|
|---|---|
|BEGIN;|BEGIN;|
|INSERT INTO accounts (acctnum, balance)<br>values (12345, 100);||
||SELECT * from accounts where acctnum = 12345;|
|ABORT;|
- Non-repeatable Reads: reading the same query results in different response (with changes to **rows that existed**), **if parallel session committed in between**

| Session A                                                   | Session B                                     |     |
| ----------------------------------------------------------- | --------------------------------------------- | --- |
| BEGIN;                                                      | BEGIN;                                        |     |
|                                                             | SELECT * from accounts where acctnum = 12345; |     |
| UPDATE accounts set balance = 500<br>where acctnum = 12345; |                                               |     |
| COMMIT;                                                     |                                               |     |
|                                                             | SELECT * from accounts where acctnum = 12345; |     |

- Phantom Reads: Reading the same query results in **new rows appeared**, **if parallel session committed in between**

|Session A|Session B|
|---|---|
|BEGIN;|BEGIN;|
|UPDATE accounts set balance = 500<br>where acctnum = 789;||
||SELECT * from accounts where balance > 0;|
|INSERT INTO accounts (acctnum, balance)<br>values (12345, 100);||
|COMMIT;||
||SELECT * from accounts where balance > 0;|
