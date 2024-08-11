`defer` annotated line placed to the call stack and will fire at the end of scope or at the end

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