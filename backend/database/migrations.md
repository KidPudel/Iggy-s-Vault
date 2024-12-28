version of control for database tables
migration is just a script (sql, bash, python) to take database from state n to state n+1


to apply changes , we need to run all migrations, and the current state of the db is tracked using version number, this way we know at what place we already are, and which migrations are already applied

This version is typically in the same db as its own table
|version|
-------------
| 0 |


# Flyway

[[flyway]]
