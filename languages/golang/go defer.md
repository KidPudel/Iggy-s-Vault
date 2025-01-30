`defer` annotated line placed to the call stack and will fire at the end of scope or at the end
first `defer` is the most "patient" one, he is deferred, until all other processes are done (since `defer` is also a process)

```go
// Function to count words in a string and update the sync.Map
func countWords(text string, wg *sync.WaitGroup, wordMap *sync.Map) {
    defer wg.Done()
    words := strings.Fields(text)
    for _, word := range words {
        wordMap.Update(word)
    }
}
```

why you sometimes need to call `defer` if it memory is automated?
because some of the resources are externally opened/utilized, and for that after you've instructed to open something, you need to close it