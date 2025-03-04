Object that defines how texture data (images [[image buffers]]) is accessed and filtered during rendering. It control how the GPU fetches texels (texture pixels) from a texture
1. filtering. How to interpolate between texels (nearest=neighbor, linear, etc.)
2. addressing. how to handle coordinates that go outside the texture bounds (wrapping, clamping)
3. and more