# python

For positional variables binding, _the second argument must always be a sequence_, even if it contains a single variable (remember that Python requires a comma to create a single element tuple):
```python
cur.execute("INSERT INTO foo VALUES (%s)", "bar")    # WRONG
cur.execute("INSERT INTO foo VALUES (%s)", ("bar"))  # WRONG
cur.execute("INSERT INTO foo VALUES (%s)", ("bar",)) # correct
cur.execute("INSERT INTO foo VALUES (%s)", ["bar"])  # correct
```

The placeholder _must not be quoted_:
```python
cur.execute("INSERT INTO numbers VALUES ('%s')", ("Hello",)) # WRONG
cur.execute("INSERT INTO numbers VALUES (%s)", ("Hello",))   # correct
```