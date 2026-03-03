Central component that constantly checks for incoming events (IO, network, timer events) and dispatches them to their coroutines.

In Python implemented via [[asyncio]], it runs asynchronous tasks and callbacks, perform network IO operations, and run subprocesses

# the process
- Encountering coroutine function call
- if it is `await` marked, then the *current* coroutine is suspended and control yielded back to the event loop to check for available coroutines to execute
	- and it is executing our coroutine, and then by finishing returns control back to the current coroutine
- if it is scheduled by creating [[asyncio Task]] or Tasks in case of `gather()`, then new task is registered in event loop and runs, meanwhile our current coroutine is not blocked and is running until `await` is called, to literally wait for the functions to finish up