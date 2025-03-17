Migrations that are not versioned, therefore applied each time when [[checksum]] is changed

Use case:
- Ideal for database objects or structures that change over time and should always be applied, regardless of the state of the database. like [[views]]
- or data that changes via configurations / data that determines behavior of the service
Useful for dynamic data or logic that is changes overtime so we don't need to add migration versions

- **Database Views**
- **Stored Procedures**
- **Functions**
- **Other Database Objects**: That may need to be recreated or updated when their definitions change.