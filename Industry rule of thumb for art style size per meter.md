# **Industry rule of thumb**

Games usually target **512–1024 px per meter** for environments. For props/UI-important items like cards, you can go higher: **1024–2048 px per meter**.

### **Calculation for your card (0.06 × 0.09 m)**

Let’s assume **1024 px/m** texel density:
  * Width: 0.06 m × 1024 px/m ≈ 61 px
  * Height: 0.09 m × 1024 px/m ≈ 92 px

That’s too small for UI-close viewing. Cards are usually zoomed in and must be sharp. So you want much higher density.

At **8192 px/m** (high detail for small props):
  * Width: 0.06 m × 8192 ≈ 492 px
  * Height: 0.09 m × 8192 ≈ 737 px


At **16k px/m** (ultra crisp for UI-focused cards):
  * Width: ~983 px
  * Height: ~1475 px

That lands in the **1024×1536 px sweet spot I suggested earlier**.
### **Conclusion**
  * If cards are seen **up close / central to gameplay** → use **1024×1536 px textures**.
  * If cards are only seen **on a table from afar** → **512×768 px** is enough.
  * Keep your art sources at 2× that size (e.g. 2048×3072) for future-proofing, but export smaller for Godot.