In golang atomics is a synchronization mechanism that is implemented with **hardware-supported atomic instructions** which ensures that the entire operation is performed in a single, individual step.
That also means that they only work on individual memory locations, meaning simple data like integers or booleans.
Also it is suited only for simple logic.