similar to [[slices under the hood]], it contains pointer to the map, but with maps doesn't store `len` and `cap` like slices, as well as maps doesn't have operations like `append()`, that exposes that it is a copy and lead to unwanted behavior.
In Golang maps are implemented using [[hash function]] that are called "hash tables"

Map is an **array of buckets**. Each bucket can hold multiple key-value pairs, using [[linked list]] under the hood. 
Hash function is used to hash a key to determine to which bucket they belong

Collisions are happening when another pair is gone to the bucket
Over time, map dynamically grows to reduce collisions

Growth strategy
When the load factor (ration of elements to buckets) exceeds a threshold, the map resizes by doubling the number of buckets and redistributing elements using a rehashes

Iteration
In map iteration is in pseudo-random order, due to internal implementation on how Go's runtime randomizes the hash seed when the program starts.


