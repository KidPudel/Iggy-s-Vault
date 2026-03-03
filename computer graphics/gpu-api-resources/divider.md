Technique used to optimize how instances ([[instancing]]) are processed by the GPU.
# 1. Divider as instance count management
When have very large number of instances like millions, we can process them by [[batching]].
Set it to 10, so it wll process 10 000 instances at a time
this can be also useful to set the specific cursor at which we can filter data with some specific logic (like mapping to the right [[transformations]] matrix)

# 2. [[Culling]] optimization
grouping instances in a grid and rendering cells that visible