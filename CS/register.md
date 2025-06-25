Small and fastest storage inside [[CPU]] with 1-cycle access, meaning CPU able to retrieve or write the data within one singe [[CPU cycle]].
Size is about 8-16 bytes
This is the place where [[CPU]] is able to do operations

- Even if you only access 1 byte, the CPU loads the entire cache line. ([[memory techniques]])
- All cache hits/misses happen **at the cache line level**.