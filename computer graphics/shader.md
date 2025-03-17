Shader is a program that runs on GPU.
The main purpose of shader is:
- process how vertices are positioned and transformed
- Determine how textures are mapped and displayed on the geometry 
- calculate light
- Apply effects 

The main point of shader is delegate the heavy computation to it from CPU
You can manage it with [[buffer]]s [[layout qualifier]]s [[binding]]s and so on and pass to it data in many many ways.

And then shader code handles one vertex -> pixel at the time, but GPU runs all of them in parallel at once, because of parallel properties of GPU.