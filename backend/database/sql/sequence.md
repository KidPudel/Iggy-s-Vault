Sequence is the special object that is single-row table created with `CREATE SEQUENCE` that is used to generate a unique identifier for row of a table. 
Basically, a self-incrementing value for example

## methods

- `nextval(regclass)`: Advances the sequence object to its next value and returns it.
- `setval(regclass, begin, [boolean])`:  sets the sequence object's current value
	- `SELECT setval('myseq', 42)`:  `nextval` will return 43
	- `SELECT setval('myseq', 42, true)`:  the same as above
	- `SELECT setval('myseq', 42, false)`:  `nextval` will return 43
- `curvar(regclass)`: returns current value 


#database