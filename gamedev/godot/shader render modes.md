
#  **cheat sheet (spatial shaders)**

In a spatial shader, you put this right under shader_type spatial;:

```
render_mode mode1, mode2, mode3, …;
```

Each mode is a keyword; you can combine several, separated by commas. The order doesn’t matter (most are independent).

  

Here are the common modes and what they control:

| **Mode**                                      | **Description / Behavior**                                                | **Use case / notes**                                             |
| --------------------------------------------- | ------------------------------------------------------------------------- | ---------------------------------------------------------------- |
| blend_mix                                     | Standard alpha blending (source.rgb * source.a + dest.rgb * (1−source.a)) | Default for translucent materials                                |
| blend_add                                     | Additive blend (color values add)                                         | For glow, fire, light effects                                    |
| blend_sub                                     | Subtractive blend                                                         | Rare, for “subtract light” effects                               |
| blend_mul                                     | Multiplicative blend (darkening)                                          | Overlays or shadow overlays                                      |
| blend_premul_alpha                            | Premultiplied alpha mode                                                  | Useful if your color textures are premultiplied                  |
| depth_draw_opaque                             | Draw depth only for opaque fragments (not transparent ones)               | If you want transparency not to write depth                      |
| depth_draw_always                             | Always write depth (even transparent parts)                               | Use carefully — can break translucency                           |
| depth_draw_never                              | Never write to depth                                                      | Good for overlays / GUI elements                                 |
| depth_prepass_alpha                           | Do an opaque-depth pass before blending transparent parts                 | Exactly what you used to stabilize order                         |
| depth_test_disabled                           | Disable depth comparison (i.e. skip depth test)                           | Forces it to draw ignoring depth ordering                        |
| cull_back                                     | Cull back faces (default)                                                 | Standard for most meshes                                         |
| cull_front                                    | Cull front faces                                                          | Useful for “inside view” meshes                                  |
| cull_disabled                                 | No face culling (double-sided)                                            | For thin planes where you want both sides visible                |
| unshaded                                      | Skip lighting and shading — just show albedo                              | Good for HUDs, overlays, or flat visuals                         |
| wireframe                                     | Render as wireframe (edges)                                               | Debugging / visualizing mesh structure                           |
| debug_shadow_splits                           | Color different shadow cascade splits differently                         | Debugging shadow cascades                                        |
| sss_mode_skin                                 | Subsurface scattering mode (skin optimization)                            | For character skin visuals                                       |
| skip_vertex_transform                         | Don’t let Godot auto-transform VERTEX / NORMAL etc                        | Use if you want to write your own vertex transform               |
| world_vertex_coords                           | Apply transform in world space instead of local                           | Use when you want to manipulate in world coordinates             |
| ensure_correct_normals                        | Helps when non-uniform scale is used                                      | Might fix normals distortions in scaled meshes                   |
| shadows_disabled                              | Shader doesn’t receive shadows (though object may still cast)             | Slight optimization when shadows aren’t needed                   |
| ambient_light_disabled                        | Don’t receive ambient light / environment light                           | Use when you override lighting fully                             |
| shadow_to_opacity                             | Lighting affects alpha (shadowed → opaque)                                | For effects like overlaying shadow on camera feed                |
| vertex_lighting                               | Use vertex-based lighting instead of per-pixel                            | Lower quality but higher performance                             |
| alpha_to_coverage / alpha_to_coverage_and_one | Enable MSAA-style alpha antialiasing                                      | Helps soften edges, where supported                              |
| fog_disabled                                  | Don’t receive fog blending                                                | Good for overlays or visual effects that shouldn’t fade with fog |

---

### **✅ Tips & combinations**

- **“Soft transparency with correct occlusion”**
    
    Use: blend_mix, depth_prepass_alpha + your ALPHA = … logic.
    
    That’s what you used and it resolved the slot peeking issue.
    
- **“Overlay / UI highlighting”**
    
    Use: depth_draw_never (and maybe depth_test_disabled) so it never blocks or is blocked. Combine with unshaded if you don’t want lighting.
    
- **“Opaque cutout style”**
    
    Use: alpha_scissor or alpha_hash (not listed above, but Godot supports them) if you want binary transparency but still participate in depth writes.
    
- **Face culling**: often cull_back, but for cards or thin quads you may prefer cull_disabled so you see both sides.
    
- **Disable lighting**: unshaded is perfect for pure color / flat display; skip all the expensive shading calc.
    
- **Debug & dev modes**:
    
    wireframe helps you see geometry, debug_shadow_splits helps track your shadow cascades.
    

---

You can use that in Obsidian as a reference. If you like, I can also create a version with just “modes I often need + examples” so it’s leaner. Do you want me to produce that too?