
[original take](https://stackoverflow.com/questions/3180375/select-vs-select-column)

There are several reasons you should never (never ever) use `SELECT *` in production code:

- since you're not giving your database any hints as to what you want, it will first need to check the table's definition in order to determine the columns on that table. That lookup will cost some time - not much in a single query - but it adds up over time
    
- if you need only 2/3 of the columns, you're selecting 1/3 too much data which needs to be retrieving from disk and sent across the network
    
- if you start to rely on certain aspects of the data, e.g. the order of the columns returned, you could get a nasty surprise once the table is reorganized and new columns are added (or existing ones removed)
    
- in SQL Server (not sure about other databases), if you need a subset of columns, there's always a chance a non-clustered index might be covering that request (contain all columns needed). With a `SELECT *`, you're giving up on that possibility right from the get-go. In this particular case, the data would be retrieved from the index pages (if those contain all the necessary columns) and thus disk I/O **and** memory overhead would be much less compared to doing a `SELECT *....` query.
    

Yes, it takes a bit more typing initially (tools like [SQL Prompt](http://www.red-gate.com/products/SQL_Prompt/index.htm) for SQL Server will even help you there) - but this is really one case where there's a rule without any exception: do not ever use SELECT * in your production code. **EVER.**