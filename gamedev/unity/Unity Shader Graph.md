# Unity Shader Graph

## Input Nodes

Position, Normal Vector, Tangent Vector, UV, Vertex Color, View Direction, Screen Position, Time, Color, Float/Vector2/3/4, Boolean, Texture 2D Asset, Sampler State, Gradient.

## Math Nodes

Add, Subtract, Multiply, Divide, Power, Square Root, Absolute, Negate, Lerp, Inverse Lerp, Smoothstep, Step, Clamp, Saturate, Min, Max, One Minus, Remap, Fraction, Floor, Ceiling, Round, Truncate, Sign, Modulo, Dot Product, Cross Product, Normalize, Length, Posterize.

## Texture Nodes

Sample Texture 2D, Sample Texture 2D LOD, Sample Texture 2D Array, Sample Cubemap, Texture Size, Texel Size, Flipbook, Tiling And Offset, Rotate, Polar Coordinates, Radial Shear, Spherize, Twirl, Triplanar.

## Artistic Nodes

Contrast, Hue, Saturation, White Balance, Blend (Burn, Dodge, Multiply, Overlay, Screen, etc.), Channel Mask, Channel Mixer, Normal Strength, Normal From Height, Normal Blend, Normal Reconstruct Z, Colorspace Conversion.

## Utility Nodes

Preview, Split, Combine, Swizzle, Fresnel Effect, Noise (Gradient, Simple, Voronoi), Checkerboard, Ellipse, Rectangle, Polygon, Rounded Rectangle, Custom Function, Sub Graph, Keyword, Branch, Comparison.

## Master Stack (URP Lit)

| Block | Type |
|---|---|
| Base Color | Color (RGB) |
| Normal (Tangent) | Vector3 |
| Metallic | Float |
| Smoothness | Float |
| Emission | Color (RGB, HDR) |
| Ambient Occlusion | Float |
| Alpha | Float |
| Alpha Clip Threshold | Float |

## Master Stack (URP Unlit)

Base Color, Alpha, Alpha Clip Threshold.

## Sources
- [Shader Graph](https://docs.unity3d.com/Packages/com.unity.shadergraph@14.0/manual/index.html)
