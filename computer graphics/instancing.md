Instancing is technique that allows to draw multiple references of the same data (**whole references of vertex buffer**) with n times of instance buffer with for example n different transformations in a single [[draw call]].
meaning: setting instance buffer step function to `PER_INSTANCE` instructs this buffer to use a single entry for all vertices of vertex buffer of that [[instance]] and so N instances of instance buffer.

NOTE: you setup number of instances explicitly, because GPU inherently know how many instances you want to draw.

we can have vertex buffer that only stores one type of quad, and instancing it and then change it with [[transformations]]

where [[indices]] is for referencing vertices, instancing references whole geometries like meshes with all their [[vertex]]ies and [[indices]].