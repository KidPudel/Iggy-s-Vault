## Input Nodes

### Texture Coordinate

Provides UV coordinates and various mapping methods for textures. Outputs include Generated, Normal, UV, Object, Camera, Window, and Reflection coordinates.

### UV Map

Accesses specific UV maps from your mesh. Use when you have multiple UV layouts.

### Geometry

Provides geometric information about the surface: Position, Normal, Tangent, True Normal, Incoming ray direction, Parametric coordinates, Backfacing detection, and Pointiness.

### Object Info

Outputs object-level data: Location, Color, Object Index, Material Index, and Random value (useful for variation across instances).

### Camera Data

Provides camera information: View Vector and View Z Depth.

### Fresnel

Calculates how much light reflects off a surface based on viewing angle. Higher values at grazing angles. Essential for realistic materials.

### Layer Weight

Similar to Fresnel but more controllable. Outputs Fresnel and Facing values, useful for edge highlights and rim lighting effects.

### RGB

Simple color picker. Outputs a single color value.

### Value

Single numerical value input. Useful for controlling parameters like roughness or metallic values.

### Tangent

Provides tangent vector information for anisotropic shading.

### Wireframe

Generates a wireframe pattern. Can be pixel or world space based.

## Output Nodes

### Material Output

The final output node. Connects to Surface, Volume, and Displacement sockets.

## Shader Nodes

### Principled BSDF

The all-in-one physically based shader. Combines multiple shader types into one node. Includes Base Color, Metallic, Roughness, IOR, Alpha, Normal, and many other parameters.

### Diffuse BSDF

Simple matte surface shader. Just diffuse reflection, no specularity.

### Glossy BSDF

Pure specular reflection. Controls: Color, Roughness, and Normal. Distribution methods: GGX, Beckmann, Ashikhmin-Shirley.

### Transparent BSDF

Allows light to pass through without refraction. Pure transparency.

### Refraction BSDF

Transparent shader with light bending (refraction). Uses IOR value to control refraction amount.

### Glass BSDF

Combines reflection and refraction for glass-like materials. More efficient than manually mixing Glossy and Refraction.

### Translucent BSDF

Light passes through and scatters, like thin cloth or leaves.

### Subsurface Scattering

Light penetrates the surface and scatters inside, then exits. Essential for skin, wax, marble, jade.

### Emission

Makes surfaces emit light. Controls: Color and Strength.

### Anisotropic BSDF

Directional specular reflection, like brushed metal or fur. Controls rotation and anisotropy amount.

### Toon BSDF

Non-photorealistic shader for cartoon-style rendering.

### Hair BSDF

Specialized shader for rendering hair and fur.

### Velvet BSDF

Simulates fabric-like materials with soft, fuzzy reflections.

### Mix Shader

Blends between two shaders using a factor (0-1). Can also use textures or Fresnel for the mix factor.

### Add Shader

Adds two shaders together. Unlike Mix, both shaders contribute fully.

## Texture Nodes

### Image Texture

Loads external image files. Supports Color Space settings, interpolation, and extension modes (Repeat, Extend, Clip).

### Noise Texture

Procedural noise pattern. Controls: Scale, Detail, Roughness, and Distortion. Useful for organic variation.

### Wave Texture

Generates wave patterns (bands or rings). Types: Bands or Rings. Profile: Sine, Saw, Triangle.

### Voronoi Texture

Creates cellular patterns. Distance metrics: Euclidean, Manhattan, Chebychev. Features: F1, F2, Distance to Edge, etc.

### Magic Texture

Creates psychedelic, abstract patterns. Primarily for abstract or stylized work.

### Musgrave Texture

Fractal noise texture. Types: Multifractal, Ridged Multifractal, Hybrid Multifractal, fBM, Hetero Terrain.

### Gradient Texture

Simple gradient. Types: Linear, Quadratic, Easing, Diagonal, Spherical, Quadratic Sphere, Radial.

### Checker Texture

Checkerboard pattern. Controls scale and two colors.

### Brick Texture

Procedural brick pattern with offset, squash, and mortar controls.

### Environment Texture

Loads HDR images for lighting. Used with World shader output.

## Color Nodes

### Mix RGB

Blends two colors using various blend modes: Mix, Add, Multiply, Subtract, Screen, Divide, Difference, Overlay, etc.

### RGB Curves

Adjust colors using curve interface. Separate controls for Combined, R, G, B channels.

### Invert

Inverts colors. Has Factor control for partial inversion.

### Hue/Saturation/Value

Adjusts hue, saturation, value, and color intensity.

### Bright/Contrast

Simple brightness and contrast adjustment.

### Gamma

Applies gamma correction to colors.

### Color Ramp

Maps input values (0-1) to a color gradient. Extremely versatile for controlling texture outputs.

## Vector Nodes

### Normal Map

Converts RGB normal map to normal vectors. Requires setting Tangent space or Object space.

### Bump

Simulates surface detail using height information. Cheaper than displacement but less accurate.

### Displacement

Actually deforms geometry. Requires subdivision or Adaptive Subdivision (Cycles).

### Vector Transform

Converts vectors between different coordinate spaces: World, Object, Camera.

### Mapping

Transforms texture coordinates: Location, Rotation, Scale. Essential for positioning and scaling textures.

### Normal

Outputs the surface normal direction or can be used to set a custom normal.

### Vector Curves

Adjust vectors using curves, similar to RGB Curves but for vector data.

### Vector Rotate

Rotates a vector around an axis.

### Vector Math

Performs mathematical operations on vectors: Add, Subtract, Multiply, Divide, Cross Product, Dot Product, etc.

## Converter Nodes

### Math

Performs mathematical operations on values: Add, Subtract, Multiply, Divide, Power, Sine, Cosine, Minimum, Maximum, Round, etc.

### Color Ramp (Converter)

Remaps value ranges to colors. One of the most useful nodes for controlling shader behavior.

### Map Range

Remaps value ranges. Input a range and output to a different range. Has Clamp option.

### RGB to BW

Converts color to grayscale value.

### Separate RGB / Combine RGB

Splits color into R, G, B channels or combines them back.

### Separate XYZ / Combine XYZ

Splits vectors into X, Y, Z components or combines them.

### Clamp

Restricts values between a minimum and maximum.

### Mix

Mixes two values using a factor. Clamped or unclamped modes.

### Wavelength

Converts wavelength value to RGB color (simulates light spectrum).

### Blackbody

Converts temperature (Kelvin) to color. Used for realistic fire and light colors.

## Tips

- **Color Ramp is your friend**: Use it to control almost any output from texture nodes
- **Math nodes are essential**: Learn to use Greater Than, Less Than, Add, Multiply for masking
- **Map Range**: Critical for converting value ranges to usable parameters
- **Principled BSDF covers 90% of materials**: Start here before using specialized shaders
- **Mix Shader with Fresnel**: Creates realistic edge highlights and layered materials
- **Bump vs Normal Map**: Bump generates normals from height; Normal Map uses pre-baked normal data
- **Noise + Color Ramp**: Quick way to create variation and masking effects