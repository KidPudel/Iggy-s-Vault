```python
@app.get("/testQuery")
async def test_query(test: str, number: int = 0):
    return [test, number]
```
	