Similar to [[ebiten fundamental concepts]] , everything that is rendered or displayed in pygame is a `Surface` object
Since its and actual surface, it is a space where the data is present, our *working screen space* is also a surface with some data.
To draw other surfaces on top of the current with the given destination (other presence (coordinate, rect or surface)), we can use methods:
- `blit`, to draw **onto current**
- `draw.<shape>`: to draw **onto target**