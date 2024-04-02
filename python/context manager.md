Context manager, is the object that defines the runtime context. It handles the entry into and the exit from the runtime context in the block of code,
1. `__enter__()`: is called when the context manager's code block entered. It performs setup and initialization required for the context
2. `__exit__()`: is called when the context manager's code block is exited. It performs the teardown or finilization.

> *protocol:* It's a way to define how the object should be set up and torn down in a controlled, managed, and consistent way.

1. Setup method is responsible for acquiring re