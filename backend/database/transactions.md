Transactions are the sequences of operations that are treated as a ***singe unit of work***, that represents a ***state of change*** on altering a database (`insert`, `update`, `delete`) and to complete transaction, we making a commit

If query failed, therefore transaction not completed, database rolls back to previous state

There is 4 key properties that define a transaction - called [[ACID]]