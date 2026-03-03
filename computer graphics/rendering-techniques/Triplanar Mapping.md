
# Triplanar Mapping

A texturing technique that projects a texture from three world-space axes (X, Y, Z) and blends them using surface normals, eliminating the need for UV unwrapping.

## What it does

- Projects the same texture from X, Y, and Z axes using world-space position as UV coordinates
- Calculates blend weights from the absolute surface normal: surfaces facing a given axis receive more of that axis's projection
- Blend weights are normalized to sum to 1 so the final color is a weighted average of all three projections
- A `sharpness` exponent applied to the normal before normalization controls how abruptly projections transition
- A surface facing straight up gets 100% of the Y projection; an angled surface blends between multiple projections
- Full triplanar samples the texture three times per fragment; top-only samples once (Y axis only)
- Texture appears to be "painted on the world" rather than the mesh — moving the object causes the texture to swim unless locked to object space

## Code

```glsl
// Projection UVs from world position
vec2 uv_x = world_pos.zy;
vec2 uv_y = world_pos.xz;
vec2 uv_z = world_pos.xy;

// Blend weights from surface normal
vec3 blend = abs(normal);
blend = pow(blend, vec3(sharpness));
blend /= (blend.x + blend.y + blend.z);

// Final color
vec3 final = texture(tex, uv_x).rgb * blend.x
           + texture(tex, uv_y).rgb * blend.y
           + texture(tex, uv_z).rgb * blend.z;
```

## Sources

- https://docs.godotengine.org/en/stable/tutorials/shaders/

## Related

- [[UV-coordinates]]
- [[normals]]
- [[fragment-shader]]
- [[shader]]

## Process

- How does normalizing the blend weights ensure the final color has the same brightness as the input texture?
- What effect does increasing the sharpness exponent have on the blending between projections?
- How does using world-space position as UVs differ from using object-space position for triplanar mapping?
- Why does full triplanar require three texture samples and how does that differ from a single UV-mapped sample in terms of GPU cost?
