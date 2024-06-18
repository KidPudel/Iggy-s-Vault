Mechanism allowing to distribute tasks across different threads or machines.

Task's queue input is the *unit of work called a **task***
Dedicated worker processes constantly monitor task queue for work/task to perform


Task queue communicates via messages, usually using [[message broker]] to mediate between clients and workers
So worker here is the other side that listens, just like in regular [[Message queue]], but instead of just some client, there is a worker