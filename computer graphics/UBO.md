Uniform Buffer Object is an efficient mechanism for passing [[uniform data]] to shaders instead of passing them separately, especially when you have a large amount of uniform data or when you need to update it frequently.
Instead of sending uniform data separately, you can group them together into a [[buffer]] and send them all at once


In order for [[shader]] to accept UBO, it needs to specify `std140` [[layout qualifier]] which speficies a std layout for the uniform block, ensuring that the shader and you CPU-side code agree on the memory layout.
But with modern approach we can specify [[binding]] directly. `binding=X`


if you need large dataset or read-write or dynamic sized array you could use [[SSBO]]


1. Read-only access
2. Supports std140 layout
3. limited to 16kb or 54 kb depending on the GPU
4. Don't support flexible/unsized array. all arrays must be fixed at compile size
5. Optimized for low latency with more limited capacity
