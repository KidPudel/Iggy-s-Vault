**Bake in Blender when:**

- **Complex procedural effects** - Voronoi, noise, complex node setups that would be expensive to calculate in real-time
- **Static objects** - Train interior walls, floors, non-interactive props
- **Unique, one-off details** - Graffiti, wear patterns, specific decorative elements
- **Ambient occlusion/lighting details** - Baked AO, cavity maps, curvature
- **Performance matters** - Mobile games, lots of objects on screen
- **You need exact consistency** - What you see in Blender is what you get in game

**Make in Godot when:**

- **Simple materials** - Solid colors, basic metal/plastic/fabric (just set metallic/roughness values)
- **Need runtime changes** - Materials that change color, transparency, or properties during gameplay
- **Reusable across multiple objects** - Generic metal, glass, concrete that many objects use
- **Tiling textures** - Repeating patterns that would waste texture space if baked per-object
- **Effects that need to be dynamic** - Glowing screens, animated shaders, water, force fields
- **Transparency/refraction** - Glass, water, hologram effects (can't bake these properly)

**For train:**

- **Bake:** Seat fabric pattern, floor tiles with wear, wall panels with detail
- **Godot:** Window glass (transparency), metal poles (simple metallic material), any LED displays or lights

The rule: If it's geometric detail or complex color patterns → bake. If it's simple PBR properties or needs to be dynamic → Godot material.