A preparation creates a *prepared* statement, that is sent to the database and waiting, when you bind unspecified values to the parameters

How it works
1. **Prepare**: An SQL statement created and sent to the database, with some values left unspecified, called parameters (?, :1, :2)
2. **Parsing and Optimisation**: Database parses, compiles, and optimising SQL template, and stores the results without executing
3. **Execute**: Later when application binds parameters and executes the prepared statement. The application then can execute this statement many times with different values.

#database