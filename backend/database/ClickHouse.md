[[columnar database]] that is best suited for [[OLAP]] scenarios.
They at least makes 

# Table Engines


## MergeTree
`MergeTree` family table engines are designed for high data ingest rates and huge data volumes.
**Insert operations create table parts, which are merged by a background processes with other table parts**
