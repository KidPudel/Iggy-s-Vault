[[CPU]] Cache is a small memory layer between [[CPU]] and [[RAM]], which is build using [[SRAM]]
- L1 - closest to the [[CPU]], very fast, very small (about 32-64 KB/core)
- L2: Larger, slower (~256 KB–1 MB).
- L3: Shared across cores, even slower (~8–64 MB).

If data is found in cache it is called [[cache hit]]
if data is NOT found in cache, then [[cache miss]] heppens


Cache doesn't store individual bytes or words. Instead, it loads and stores blocks of memory called [[cache line]]