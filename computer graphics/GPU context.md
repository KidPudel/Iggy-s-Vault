GPU context is a connection between GPU and running program.
- It manages graphics state like tracking active shaders, textures, buffers and other GPU resources.
- Handles/propagates commands through the connection between GPU and CPU

The example of 2D context is SDL_Renderer in [[SDL]], it is responsible for managing the context and the whole graphics pipeline

GPU Device in SDL3 which will replace the renderer, and let you implement your own renderer
