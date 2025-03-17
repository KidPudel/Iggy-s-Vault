Event bus is an implementation of pub/sub [[message patterns]] that allows different components to communicate without direct dependencies.
- Components doesn't know about each other
- Events can be processed independently and in parallel [[concurrency in go]]

```go
type EventBus struct {
	listeners map[string][]func(data string)
}

func NewEventBus() *EventBus {
	return &EventBus{
		listeners: make(map[string][]func(data string))
	}
}


func (bus *EventBus) Sub(eventType string, callback func(data string)) {
	if _, ok := bus.listeners[eventType]; !ok {
		bus.listeners[eventType] = make([]func(data string), 0)
	}
	bus.listeners[eventType] = append(bus.listeners[eventType], callback)
}

func (bus *EventBus) Emit(eventType, data string) {
	for _, callback := range bus.listeners[eventType] {
		callback(data)
	}
}
```


```go
func trackOvversForX(data string) {
	fmt.Pritnln("X tracking: ", data)
}

func trackOffersForY(data string) {
	fmt.Println("Y tracking: ", data)
}

func main() {
	bus := NewEventBus()
	bus.Sub("offers", trackOffersForX)
	bus.Sub("offers", trackOffersForY)

	// some executions
	...
	bus.Emit("offers", currentOffers)
}
```