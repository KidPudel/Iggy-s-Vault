Shader Storage Buffer Object like [[UBO]], but it has
1. Read/Write access, that persists across [[shader]] invocation
2. Supports std140 and std430 layout
3. Allowing storing gigabytes of data. making them suitable for large datasets
4. Supports flexible/unsized array
5. More dataset, but higher access latency



```glsl
layout(std430, binding = 0) buffer MvpData {
	vec4 mvps[];
}
```


In order for [[shader]] to accept SSBO, it needs to specify `std140` or `std430` [[layout qualifier]] which specfies a std layout for the uniform block, ensuring that the shader and you CPU-side code agree on the memory layout.
But with modern approach we can specify [[binding]] directly. `binding=X`

Since it is a buffer, we should add it to our [[binding]]s as a storage buffer.

So it could be used as more flexible way of passing even vertex buffers, without having to define [[attribute data]] in layout

Instead of updating [[uniform data]] every [[draw call]], you upload once and reuse the storage buffer across frames



**Increased CPU Overhead (Updating the VBO Every Frame)**

- **If the text changes or moves**, you must **recompute and re-upload** the entire vertex buffer.
    
- This is **slower** than just updating an SSBO because:
    
    - VBO updates require **buffer reallocation or orphaning**.
        
    - SSBOs can be updated in **chunks** (e.g., `glMapBufferRange()`).


### **1️⃣ Dynamic VBO (Precompute on CPU)**

💡 **Idea:** Store **fully transformed vertex positions** in the VBO (instead of a generic quad) and update it whenever text changes.

✅ **Advantages:**

- **Reduces per-instance shader work** (transforms are done on the CPU).
    
- **No need for instancing**—just use `glDrawElements()`.
    
- **Easier to integrate with CPU-side text layout engines**.
    

❌ **Disadvantages:**

- **CPU overhead**: Every time text changes, the entire VBO needs to be **rebuilt and re-uploaded**.
    
- **Higher memory bandwidth usage**: More data sent to the GPU compared to just a **small SSBO update**.
    
- **Slower for dynamic text updates**.
    

---

### **2️⃣ Instancing (Static Quad + Transform in Shader)**

💡 **Idea:** Use **one static quad** in a VBO and transform it dynamically in the **vertex shader**, using **instancing or SSBO**.

✅ **Advantages:**

- **Minimal CPU work**: No need to recompute and upload new vertex data every frame.
    
- **Efficient for large text buffers**: Only instance data (position, scale, UV) needs updating.
    
- **Memory efficient**: No need to store large amounts of transformed vertex data.
    

❌ **Disadvantages:**

- **Per-instance transformation in the shader** (which might be slightly more expensive).
    
- **Shader execution per-vertex does scaling and positioning**.



---


- **For UI elements / small text (menus, labels)** → **Dynamic VBO is OK**
    
- **For large, dynamic text (editors, terminals)** → **Use instancing + SSBO**