Input is a [[vertex]] [[buffer]] which contains all vertices for every triangle to form a 3D model.

Since it comes in a big chunk, we can't distinguish data. For that we tell input assembler which attributes make up vertex.
- position: xyz
- [[UV coordinates]]
- [[normals]]
- ID

(4.55, 3.33, 5.03), (0.09, 0.56), (0.50, 0.73, 0.91), 0,
(1.35, 3.63, 5.03), (0.09, 0.56), (0.50, 0.73, 0.91), 1,
(8.51, 3.33, 5.03), (0.09, 0.56), (0.50, 0.73, 0.91), 2,
(1.15, 3.33, 5.03), (0.09, 0.56), (0.50, 0.73, 0.91), 3,
(6.01, 3.33, 5.03), (0.09, 0.56), (0.50, 0.73, 0.91), 4,
(1.00, 3.33, 5.03), (0.09, 0.56), (0.50, 0.73, 0.91), 5,