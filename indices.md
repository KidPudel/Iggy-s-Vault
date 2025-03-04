Numerical values that references to vertices ([[vertex]]) in a 3D model's vertex array.
So instead of storing complete vertex data multiple times, indexes point to existing vertices

```
vertices: [v0, v1, v2, v3]
indices: [0,1,2, 0,2,3]
```

This way by using 4 vertices, we've created 2 triangles and therefore saved memory, rendered faster