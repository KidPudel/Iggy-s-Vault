**Frame**/Screen/Video [[buffer]] is the collection of bound [[image buffers]] (memory chunks) **to which we perform drawing operations** on the GPU.
For that you use shader programs ([[vertex shader]] and [[fragment shader]]). And shader program needs to know into which chunk they will draw, aka into which image.


"So far we've used several types of screen buffers: a color buffer for writing color values, a depth buffer to write and test depth information, and finally a stencil buffer that allows us to discard certain fragments based on some condition. The combination of these buffers is stored somewhere in GPU memory and is called a framebuffer."


So it could be a rendering target.