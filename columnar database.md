Columnar or column-orientated representation is [[DBMS]] that stores data in columns instead of rows.
It makes reading data or aggregating queries (sum, average, max, min) very quickly, by **instead of reading all of columns from left to right like in row-based orientation, we read only necessary columns from top to bottom**
And in this orientation, this system is acknowledged of which data is corresponding to which column, by assigning a number to each row of data
It is also highly recommended to order the data, allowing for much faster queries

But this is makes inserting new row somewhere in the middle very slow