Normalized Device Coordinates are coordinates in 3D space between -1 and 1 that are processed in the [[vertex shader]], and they should be normalized by [[transformations]]
If they fall outside this range, they will be discarded/clipped and won't be visible on your screen


Unlike usual coordinates 0, 0 coordinates is a center

![[Pasted image 20250302142704.png|500]]

- The x-axis ranges from -1 (left) to +1 (right)
- The y-axis ranges from -1 (bottom) to +1 (top)
- The z-axis ranges from -1 (near) to +1 (far) in OpenGL, or from 0 (near) to 1 (far) in Direct3D


```c
float vertices[] = { 
   -0.5f, -0.5f, 0.0f,
	0.5f, -0.5f, 0.0f,
	0.0f,  0.5f, 0.0f
};
```



# convert screen-space pixel values to NDC
## X-Axis conversion (left-to-right)
- Normalize coordinates by dividing by screen width
- Then scale by 2.0 because we normalized screen coordinates (NSC) gave us only a right half of NDC (only positive values), so we double it.
- and shifting value by -1
## Y-Axis conversion (Flipping to OpenGL's Bottom-to-Top)
- Normalize
- Then scale by 2.0
- Because OpenGL's y-axis is flipped (y=0 top, but in NDC y=1 top), we flipping subtraction of `1 - value`


