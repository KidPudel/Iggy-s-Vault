In python coroutine is a more generalized form of ***subroutines***, that are entered at one point and exited at another point, that are returned by the function with `async/await`
#### 👎👎👎
> **NOTE**: Simply calling coroutine function `coroutine_1()`  ==**WILL NOT**== schedule/register it to be executed with the event loop, it will simply return the coroutine object and not run it, and you need to explicitly put it to the event loop

#### How you should do it.
So to run the top-level coroutine entry point, use `run()`.
```python
asyncio.run(main())
```
This function runs the passed coroutine and takes care of managing [[asyncio]]'s [[Event loop]], *finalizing asynchronous generators, and closing the executor*
> **NOTE**: this function cannot be called when another asyncio event loop is running



> The rest of coroutines could be launched either sequentially with `await`, or concurrently with [[asyncio Task]]
## awaiting
```python
async def current_coroutine():
	print("before await")
	await sending_request()
	print("first is done")
	await send_another_request()
	print("after all awaits")
```
When encountering `await`, current coroutine is **suspended** and control yields back to the event loop, to check on other coroutines and I/O executions.
Here event loop starts to execute `sending_request()` coroutine
after the coroutine is done, current coroutine is resumed and continues to execute, and can encounter the next `await`


## Concurrent running
### [[asyncio Task]]
```python
async def current_coroutine():
	task1 = asyncio.create_task(coroutine_1())
	task2 = asyncio.create_task(coroutine_2())
	print("both coroutines started concurrently")
	# only at this point we await it
	await task1
	await task2
	print("bot finished")
```

or with modern `TaskGroup`
```python
async def current_coroutine():
	async with asyncio.TaskGroup() as tg:
		task1 = tg.create_task(coroutine_1())
		task2 = tg.create_task(coroutine_2())
		print("both coroutines started")
	# await is implicit
	print("both finished")
```


### gathering
`asyncio.gather(*aws, return_exceptions=False)` run awaitable `*aws` sequence concurrently
To note:
- If any of `aws` is coroutine, it is automatically scheduled as a [[asyncio Task]]
- If all completed successfully, it returns a list of returned values
- if `return_exception=False`, even if one of coroutines is raised exception, others **won't be cancelled**
- if `return_exception=True`, exception list is treated the same as successful results, and aggregated in the result list
- if `gather()` is cancelled, all coroutines are also cancelled (that hasn't finished)

```python
async def current_coroutine():
	result_list = await asyncio.gather(coroutine_1(), coroutine_2())
	# [2, 6] for example
```
