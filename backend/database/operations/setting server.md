host name or address, in context of database *is the address to which you want to point to access database*, 
so for example if you want to call database when hosting on your own machine, you need to set up a [[localhost]] (172.0.0.1), 
or if you for example running db in a docker container, then you need to specify an address of a base image, where it running on.
### side note from chat-gpt
**Production:**
- In a production environment, it's common to use a dedicated database server.
- The database server may have a different hostname or IP address that you should specify in your connection string.
- Avoid using "localhost" in production connection strings unless the database server is actually on the same machine as your application, which is uncommon in production setups.


# Beginning
To start working with Postgresql, we need to install it, as well if we want to work with user interface, we need to install pgAdmin.  
To install postgresql, on a mac, we will use a homebrew
```bash
brew install postgresql
```

After postgreqsql successfully installed, we for the first time can interact with postgresql via cli.   

Firstly, we need to start our postgresql service. 
```bash
brew services postgresql@14
```
After that, we need to create our role that we want to use, to connect to server
```sql
select * from pg_roles;
create role postgres with login password 'postgres';
```

Now we are ready to create our first **server**!!!  

and then creating database
```sql
CREATE DATABASE chinesebee_db
    WITH
    OWNER = postgres
    ENCODING = 'UTF8'
    LC_COLLATE = 'C'
    LC_CTYPE = 'C'
    TABLESPACE = pg_default
    CONNECTION LIMIT = -1
    IS_TEMPLATE = False;
```

#database