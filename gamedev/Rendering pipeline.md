In many rendering libraries, the process of rendering graphics is happening as follows:
1. **Preparing** the rendering context and drawing [[buffers]] for the following frame.
2. **Clearing the background**, which is typically the first step in the rendering new frame. This function fills entire rendering area (window or [[framebuffer]]) with solid colour. It is done to avoid previous frame to appear on the next, which leads to artifacts.
3. **Rendering/drawing the content**.
4. **End of drawing**. Finalisation of the drawing for the current frame. Swapping the back and front [[buffers]] (if using double buffering), making the rendered frame visible on the screen.