enabled [[cache hit]], low latency, [[SIMD]]

# Summary
- Use the smallest data types possible
- Store data by locality of use
- Don't waste the cache line



techniques:
- [[hot and cold data separation]]
- [[SoA over AoS]] for [[SIMD]]
- Field reordering to improve packing in [[cache line]] and avoids padding
- [[Out of band]]

