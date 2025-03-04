Mathematical operations that change position, orientation, scale, or other properties of object in a virtual screen.
At its code, a transformation is a way of converting from one coordinate system to another.
We use matrices to represent and apply these transformations. When multiplied with a coordinate vector it produces a new coordinate system.

# Transformation pipeline
## 1. Model Space
Where we start defining our vertices
This space is also called *local space* or *object space*
They describe where points (vertices) are located relative to object's own center or origin.
I would also describe **self space**

## 2. World Space
First transformation applies to model position *model matrix* to convert from *model* space to world space.
Model matrix is:
- Translation (moving the object in space)
- Rotation (turning the object)
- Scaling (changing object's size)
```c
world_positon = model_positon * model_matrix
```

## 3. View Space
Next, we apply *view matrix* to transform world space to the view space / camera space
View matrix represents camera's position and orientation. It practically shifts the whole world so that "camera" would be at the origin!
```c
view_positon = world_positon * view_matrix
```

## 4. Projection Space
Next, we apply *projection matrix* to transform from view (camera) space to clip space.
This will configure and adjust how 3D coordinates are *projected* into 2D place / screen.
It handles:
- Perspective (making distant objects appear smaller)
- Filed of view (how wide the camera can see)
- Near and far planes (min and max visible distance)
```c
projection_space = view_space * projection_matrix
```


## 5. Perspective Division
After, we perform perspective division by dividing x, y, z by `w` coordinate (look at [[homogeneous coordinates]]), which was affected by the projection matrix.
This creates the perspective effect where distant objects appear smaller. **And** the result is [[NDC]], universal format.
```c
ndcPosition.x = clipPosition.x / clipPosition.w
ndcPosition.y = clipPosition.y / clipPosition.w
ndcPosition.z = clipPosition.z / clipPosition.
```

## 6. Viewport Transformation
Finally, we transform [[NDC]] to the screen coordinates (actual pixels of the screen)!
```c
screen_x = (ncd_x + 1) * viewport.width / 2 + viewport.x
screen_y = (ncd_y + 1) * viewport.height / 2 + viewport.y
```



