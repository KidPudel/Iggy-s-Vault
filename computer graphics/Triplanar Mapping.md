
## What It Is

Triplanar mapping projects textures onto a mesh from **three axes** (X, Y, Z) and blends them based on surface normals. It eliminates the need for UV unwrapping.

---

## The Problem It Solves

Traditional UV mapping has issues with:

- **Stretching** on steep surfaces
- **Seams** at UV boundaries
- **Time cost** of manual unwrapping
- **Distortion** on organic/terrain meshes

Triplanar fixes all of these by ignoring UVs entirely.

---

## How It Works

```
1. Get the world position of the pixel
2. Project texture from X, Y, and Z axes using world position
3. Blend the three projections based on surface normal direction
```

### Visual Concept

```
        Y (top-down)
        ↓
        ┌───────┐
       ╱       ╱│
      ╱   ●   ╱ │   ← Z (front)
     ┌───────┐  │
     │       │  │
     │   ●   │ ╱
     │       │╱
     └───────┘
         ↑
         X (side)
```

A pixel on a **horizontal surface** gets most of its color from the Y projection. A pixel on a **vertical wall** gets color from X or Z projection. **Angled surfaces** blend between projections.

---

## The Math

### Projection UVs

```glsl
vec3 world_pos = get_world_position();

vec2 uv_x = world_pos.zy;  // Project from X axis (side)
vec2 uv_y = world_pos.xz;  // Project from Y axis (top)
vec2 uv_z = world_pos.xy;  // Project from Z axis (front)
```

### Blend Weights

```glsl
vec3 blend = abs(normal);
blend = pow(blend, vec3(sharpness));  // Higher = sharper transitions
blend /= (blend.x + blend.y + blend.z);  // Normalize to sum of 1
```

|Normal Direction|blend.x|blend.y|blend.z|Result|
|---|---|---|---|---|
|(0, 1, 0) up|0|1|0|100% Y projection|
|(1, 0, 0) side|1|0|0|100% X projection|
|(0.7, 0.7, 0)|0.5|0.5|0|50% X, 50% Y blend|

### Final Color

```glsl
vec3 color_x = texture(tex, uv_x).rgb;
vec3 color_y = texture(tex, uv_y).rgb;
vec3 color_z = texture(tex, uv_z).rgb;

vec3 final = color_x * blend.x + color_y * blend.y + color_z * blend.z;
```

---

## Godot Implementation

### Full Triplanar (3 projections)

```gdshader
shader_type spatial;

uniform sampler2D albedo_texture : source_color, filter_linear_mipmap, repeat_enable;
uniform float texture_scale : hint_range(0.01, 1.0) = 0.1;
uniform float blend_sharpness : hint_range(1.0, 16.0) = 4.0;

void fragment() {
    vec3 world_pos = (INV_VIEW_MATRIX * vec4(VERTEX, 1.0)).xyz;
    vec3 world_normal = abs(NORMAL);
    
    // Calculate blend weights
    vec3 blend = pow(world_normal, vec3(blend_sharpness));
    blend /= (blend.x + blend.y + blend.z);
    
    // Sample from three axes
    vec3 col_x = texture(albedo_texture, world_pos.zy * texture_scale).rgb;
    vec3 col_y = texture(albedo_texture, world_pos.xz * texture_scale).rgb;
    vec3 col_z = texture(albedo_texture, world_pos.xy * texture_scale).rgb;
    
    // Blend
    ALBEDO = col_x * blend.x + col_y * blend.y + col_z * blend.z;
}
```

### Top-Only (optimized for terrain)

```gdshader
shader_type spatial;

uniform sampler2D albedo_texture : source_color, filter_linear_mipmap, repeat_enable;
uniform float texture_scale : hint_range(0.01, 1.0) = 0.1;

void fragment() {
    vec3 world_pos = (INV_VIEW_MATRIX * vec4(VERTEX, 1.0)).xyz;
    ALBEDO = texture(albedo_texture, world_pos.xz * texture_scale).rgb;
}
```

---

## Performance

|Method|Texture Samples|Use Case|
|---|---|---|
|UV mapping|1|Simple meshes with good UVs|
|Top-only triplanar|1|Terrain, floors|
|Full triplanar|3|Cliffs, rocks, organic shapes|

> [!tip] Optimization Use top-only for terrain. Only use full triplanar on meshes with steep surfaces where stretching is visible.

---

## Common Issues

### Texture Looks Too Big/Small

Adjust `texture_scale`. Lower = larger texture, higher = smaller/more tiled.

### Visible Seams at Blend Boundaries

Increase `blend_sharpness` for harder transitions, or decrease for softer blending.

### Texture Swimming When Object Moves

Use `MODEL_MATRIX` instead of `INV_VIEW_MATRIX` to lock texture to object:

```gdshader
vec3 world_pos = (MODEL_MATRIX * vec4(VERTEX, 1.0)).xyz;
```

### Different Texture on Each Axis

Intentional feature—use different textures per axis:

```gdshader
vec3 col_x = texture(side_texture, world_pos.zy * scale).rgb;
vec3 col_y = texture(top_texture, world_pos.xz * scale).rgb;
vec3 col_z = texture(side_texture, world_pos.xy * scale).rgb;
```

---

## When to Use

|Scenario|Use Triplanar?|
|---|---|
|Terrain|✅ Yes (top-only)|
|Rocks/cliffs|✅ Yes (full)|
|Walls/buildings|❌ No, UV is fine|
|Characters|❌ No, need precise UV control|
|Procedural geometry|✅ Yes|
|Imported meshes without UVs|✅ Yes|

---

## Related Topics

- [[UV coordinates]]
- [[Godot Shaders]]
- [[normals]]
- [[PBR Texturing]]

---

## References

- [Godot Shader Documentation](https://docs.godotengine.org/en/stable/tutorials/shaders/)
- [Ben Golus - Normal Mapping for Triplanar](https://bgolus.medium.com/)