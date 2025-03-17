Draw call is the process that involves data that you setup in rendering state
In CPU:
- Binds [[shader]] program
- Binds [[VAO]]
- Binds [[texture]]
- Sets [[uniform data]]
Then it sends calls to the GPU
In GPU:
- Vertex fetch from [[VBO]] specified in [[VAO]]
- [[graphics pipeline]]


> NOTE: and then for the next draw call it passes next set of rendering state. but future frames will be optimized/cached for the **certain** data that hasn't changed.


When to use multiple draw calls
- Different shaders
- Different vertex format like attribute layout
**When to Avoid Multiple Draw Calls (If Possible):**
1. **Same Shader, Texture, and Rendering State:** If you have multiple objects that can be rendered using the same shader, texture, and rendering state, you should try to combine them into a single draw call.
2. **Large Numbers of Objects:** If you have a large number of objects in the scene (e.g., hundreds or thousands), the overhead of issuing a separate draw call for each object can become significant.
3. **Performance Bottleneck:** If your application is already CPU-bound (meaning the CPU is the limiting factor), the overhead of multiple draw calls can exacerbate the problem.

To minimize draw calls we need to utilize single state
Putting textures to atlases or bitmaps to refuce data
And make no draw calls when static 