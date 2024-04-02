Cursor is the structure (object), that allows to traverse through a data. It acts/serves as a pointer to the row, that allows to read that data


Database cursor is the defined database object that acts like a pointer, that allows as to read the records of the query result.

The base process
- SQL query gets executed and the result set containing records is stored in database server
- cursor *fetches* the result set (one at the time, or a whole bunch), by sending the request to the database server, and returning the result
- cursor then shifts to the next record (if any), since cursor keeps track of where it should be.
- close the cursor, to release the database resources