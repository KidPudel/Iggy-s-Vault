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

NOTE: setting foreign key to another field (even primary id of another table) does improve performance by itself so create your own index if needed
```sql
CREATE INDEX idx_dimension_timelines_element_id ON dimension_timelines(element_id);
```
