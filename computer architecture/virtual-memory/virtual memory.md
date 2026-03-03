Virtual memory is a layer of abstraction, where each process is given the illusion of having its own large, private, contiguous memory space - regardless of how much [[RAM]] we physically have,
each virtual address is mapped to the physical memory address, and the process does not care about physical one.

This approach allows for simpler programming model, where program assumes it has contiguous memory
```c
int arr[1000];
```
but it has not. but it doesn't matter.

# Reservation
Before mapping, we first need to reserve virtual memory. OS marks virtual memory as being used. Now the reserved chunk of virtual memory that the process is going to access in the future is reserved, [[virtual memory page]] is created.
NOTE: no physical [[RAM]] is involved. The memory is unbacked until used
Example: `malloc()` which reserves virtual memory. (so it would be more clear to call it `mreserv()`)
# Physical memory allocation and virt memory mapping
Happens on the first data access (typically write), which triggers [[Page Fault]]