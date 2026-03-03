In general use CPU for one-time or infrequent computation
- pre-computations
- caching
- asset loading
- font [[rasterization]]
- creating something
- screen space to [[NDC]] conversion
- upload data to GPU

GPU for per-frame, per-vertex, or per-pixel computation
- rendering
- transformations
- lighting
- effects
- [[transformations]], applied per vertex and changes frequently
- Lightning calculation. per-pixel
- animations, should be applied per frame

1. Does the computation need to be done for each frame?
    - **Yes →** **GPU** (Rendering, transformations, lighting, effects).
    - **No →** **CPU** (Precomputations, caching, asset loading).
2. Does the task involve many small, independent calculations?
    - **Yes →** **GPU** (Parallel processing like shaders).
    - **No →** **CPU** (Sequential logic like game state updates).
3. Does it require frequent updates to GPU memory?
    - **Yes →** Consider **CPU pre-processing** to avoid bottlenecks.
4. Can the data be reused across frames?
    - **Yes →** Compute once in **CPU**, upload to GPU, reuse.
    - **No →** Compute dynamically in **GPU** (like real-time effects).
