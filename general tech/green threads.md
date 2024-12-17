Green [[thread]] or user thread that are managed **entirely** by the [[runtime]], rather than the operating system (OS)
Green threads emulate multithreaded environments without relying on any native OS abilities.

Traditional green threads often rely on **cooperative scheduling**, where threads explicitly yield control
They could block entire runtime with [[syscalls]], because kernel assumes it deals with a single thread

all threads share the same context for entire [[runtime]]
