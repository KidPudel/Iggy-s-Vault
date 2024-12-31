General-purpose computations, returning values like number, strings, booleans, rows, sets of rows
Can be invoked in queries (`SELECT`, `UPDATE`,`INSERT`)
Process that could be executed using `EXECUTE FUNCTION` for example in a [[trigger]]
```
CREATE [ OR REPLACE ] FUNCTION
    _`name`_ ( [ [ _`argmode`_ ] [ _`argname`_ ] _`argtype`_ [ { DEFAULT | = } _`default_expr`_ ] [, ...] ] )
    [ RETURNS _`rettype`_
      | RETURNS TABLE ( _`column_name`_ _`column_type`_ [, ...] ) ]
  { LANGUAGE _`lang_name`_
    | TRANSFORM { FOR TYPE _`type_name`_ } [, ... ]
    | WINDOW
    | { IMMUTABLE | STABLE | VOLATILE }
    | [ NOT ] LEAKPROOF
    | { CALLED ON NULL INPUT | RETURNS NULL ON NULL INPUT | STRICT }
    | { [ EXTERNAL ] SECURITY INVOKER | [ EXTERNAL ] SECURITY DEFINER }
    | PARALLEL { UNSAFE | RESTRICTED | SAFE }
    | COST _`execution_cost`_
    | ROWS _`result_rows`_
    | SUPPORT _`support_function`_
    | SET _`configuration_parameter`_ { TO _`value`_ | = _`value`_ | FROM CURRENT }
    | AS '_`definition`_'
    | AS '_`obj_file`_', '_`link_symbol`_'
    | _`sql_body`_
  } ...
```

Regular function accepts parameters like `(arg int, arg2 boolean)`
Returns a value of a specified type (e.g., `INTEGER`, `TEXT`, `TABLE`, etc.). like `RETURNS INT`

To calculate the result in a function we should tell PostgreSQL where the function starts, we do it with a **delimiter** with any characters, but common is `$$`.
and inside that we could define a scope of the procedural logic of the function scope with a `BEGIN` and `END;`