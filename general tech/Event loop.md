Central component that constantly checks for incoming events (IO, network, timer events) and dispatches them to their coroutines.

# the process
1. When encountering asynchronous function, event loops adds it to the tasks, to also execute
2. Starts executing
3. Other async function could be added to the task
4. When encountering the `await` it yields the control of the process back to the event loop to continue processing other tasks
5. When second await is encountered control yielded back again to the event loop
6. it process other tasks
7. then when await completes, event loop get notified about it and it proceed execution of the task
8. then those tasks are finished itself and event loop get notified again, that it can proceed the execution of in our case the main function (or any outer outer function)
9. outer function proceeded and maybe finished