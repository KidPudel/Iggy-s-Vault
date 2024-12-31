special [[function]] that is used only as part of a trigger, it is executes **automatically** when specified event is occurs that you've defined in a [[trigger]]

It is interacts with `NEW` and `OLD` pseudo records

Invocation
```sql
CREATE TRIGGER my_trigger
AFTER UPDATE ON my_table
FOR EACH ROW
EXECUTE FUNCTION my_trigger_function();
```

It does not accept explicit parameters, instead it operates on `NEW` and `OLD`
It must return a `row` or `null`, where `NULL` is to skip operation
Return type is always `RETURNS TRIGGER`

```sql
CREATE FUNCTION check_config_version()
RETURNS TRIGGER AS $$ -- function scope
BEGIN
	IF NEW.version != OLD.version + 1 THEN
		RAISE EXCEPTION 'new version must be greater than old by 1';
	END IF;
	-- allow new record
	RETURN NEW;
END;
$$ LANGUAGE plpgsql;
```
