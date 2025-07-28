"Apply" concept means take the metadata-level transformations (like transforms, modifiers, parenting, constraints) pending and [[bake]] their results into the object's base data ([[mesh]], [[vertex]]), finalizing it and making it a new source of truth (new canonical state).

It does **not** always mean _baking into image data_ (like textures or lighting). That’s a **separate** process (baking textures, lightmaps, AO, etc.).

- **Transforms**: Object matrix (location, rotation, scale) is zeroed/normalized, and the **mesh is updated** to preserve visual appearance.
- [[modifiers]]: The non-destructive modifier is **collapsed** into permanent geometry changes (geometry is rewritten).
- **Parenting/Constraints**: Applying clears the parent or constraint, but **keeps the final pose**.


What is not included in “apply”
- Material baking
- Lighting baking
- Texture baking
These are **explicit baking processes** using image outputs, not part of the Apply system.



“Apply” in Blender = finalize and bake metadata into mesh/object data.
You use it when:
- You want [[modifiers]] to **behave consistently**.
- You’re preparing **assets for export**.
- You need **clean transforms** (0/0/0 and 1/1/1).