`Task` is the object that runs [[asyncio coroutines]] in the event loop
Tasks are used to schedule coroutines ***concurrently***

We can create a `Task` using `create_task()` which is also will schedule its execution.


> **IMPORTANT**: Save a reference to the result of the `create_task` function, to avoid task disappearing mid-execution.
> The event loop creates only weak references to task, so the task that isn't references elsewhere may get garbage collected before it's done. For reliable "fire-and-forget" background tasks, gather them in a collection

```python
background_tasks = set
for i in range(10):
	task = asyncio.create_task(some_coro(param=i))
	background_tasks.add(task)
	task.add_done_callback(background_tasks.discard)
```


> NOTE: there is also a `Future` object, which is a special low-level awaitable object that represents an **eventual result** of an asynchronous operation