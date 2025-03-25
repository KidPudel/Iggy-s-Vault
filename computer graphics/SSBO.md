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


In order for [[shader]] to accept SSBO, it needs to specify `std140` or `std430` [[layout qualifier]] which speficies a std layout for the uniform block, ensuring that the shader and you CPU-side code agree on the memory layout.
But with modern approach we can specify [[binding]] directly. `binding=X`

Since it is a buffer, we should add it to our [[binding]]s as a storage buffer.

So it could be used as more flexible way of passing even vertex buffers, without having to define [[attribute data]] in layout

Instead of updating [[uniform data]] every [[draw call]], you upload once and reuse the storage buffer across frames