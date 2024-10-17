[[primary key]] that is referenced in other table, to represent the table
```sql
CREATE TABLE reference_receipts (
    id UUID NOT NULL UNIQUE,
    type INT NOT NULL,
    CONSTRAINT fk_receipt
        FOREIGN KEY(id)
            REFERENCES receipts(id)
            ON DELETE CASCADE
);
```