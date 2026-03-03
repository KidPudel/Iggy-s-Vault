# Go Map Under the Hood

Map is an array of buckets. Each bucket holds multiple key-value pairs using a linked list. A hash function maps keys to buckets.

Collision: multiple keys landing in the same bucket → linked list grows.

Growth: when load factor (elements/buckets) exceeds threshold, map doubles buckets and rehashes everything.

Iteration order is pseudo-random — Go runtime randomizes the hash seed on startup by design.

Unlike slices, maps don't expose length and capacity separately, and have no append — so the copy-on-write footgun doesn't apply.

https://go.dev/blog/maps
