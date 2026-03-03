# sync.RWMutex

Reader/writer lock. Multiple goroutines can hold `RLock` simultaneously; only one can hold `Lock`.

- `Lock` / `Unlock` — exclusive write access
- `RLock` / `RUnlock` — shared read access

https://pkg.go.dev/sync#RWMutex
