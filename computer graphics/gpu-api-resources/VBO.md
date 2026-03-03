Vertex Buffer Object is feature that allows you to store data directly on the high-speed memory of the GPU.
So it is a container for vertex data ([[attribute data]]) per vertex or per instance, that is optimized for fast access by the GPU to easily reuse data to render.
It is bound to the [[VAO]], which specifies how the GPU should interpret the data.

You can create static or dynamic VBOs, which will optimize memory allocation and delivery process.

f If changes frequently, can become a bottleneck