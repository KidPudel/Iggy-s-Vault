Zig language performs no memory management on behalf of the programmer.
This is why Zig has not [[runtime]], and why Zig code works seamlessly in so many environments, including real-time software, OS kernels, embedded devices, and low latency servers.

Unlike C, Zig doesn't has a default allocator, like `malloc`, `realloc`, and `free`
We can use this allocator `std.heap.c_allocator`, however, by convention, there is no default allocator in Zig. So instead of this, all functions that needs to allocate something, accepts an `Allocator`, for the programmer to choose and pass.
The example of this would be initializing ArrayList [[Data structures]] `std.ArrayList`

# Choosing Allocator
- If you are making a library, the best is to accept an `Allocator` as a parameter to allow user's of library to decide
- If the maximum number of bytes that you'll need to allocated is bounded to known at compile time [[comptime]], then use `std.heap.FixedBufferAllocator` or `std.heap.ThreadSafeFixedBufferAllocator`
- Linking libc? `std.heap.c_allocator`
- Your program is command line application without fundamental cyclical pattern (game loop, web server request handler), then use [[arena allocator]] with this pattern `std.heap.ArenaAllocator.init(std.heap.page_allocator);`
- If opposite case, where you have cyclical pattern, then **and** allocations can all be freed at the end of the cycle, then [[arena allocator]] is great option, or if we have the upper bound established, then we could use `std.heap.FixedBufferAllocator` for further optimizations
- Writing a test? `std.testing.FailingAllocator`
- Finally, if none of the above apply, you need to use general purpose allocator, `std.heap.GeneralPurposeAllocator`,