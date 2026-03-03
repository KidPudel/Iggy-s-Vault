Simple and efficient allocator that manages the [[arena]], from which allocations are made.
It's designed to allocate memory in a fast and straightforward manner, via linear allocation, **without the individually deallocation**, but rather free all memory at once

# Linear allocation
When you allocate a memory, allocator moves a pointer forward in a pre-allocated memory block with a size of memory you've specified. This makes allocation O(1), constant time, since it's just involves pointer arithmetic.