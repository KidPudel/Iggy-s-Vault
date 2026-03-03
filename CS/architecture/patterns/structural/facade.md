# Facade

A structural design pattern that provides a simplified interface to a complex subsystem.

## What it does

A facade wraps a set of subsystem classes behind a single, unified interface. Clients interact only with the facade; the subsystem classes are not exposed. The facade coordinates the subsystem calls in the correct order.

The subsystem is not replaced — it remains accessible directly if needed.

## Code

```csharp
// Complex subsystem
public class VideoDecoder { public void Decode(string f) { } }
public class AudioDecoder { public void Decode(string f) { } }
public class Display { public void Render() { } }

// Facade
public class VideoPlayerFacade {
    private readonly VideoDecoder _video = new();
    private readonly AudioDecoder _audio = new();
    private readonly Display _display = new();

    public void Play(string file) {
        _video.Decode(file);
        _audio.Decode(file);
        _display.Render();
    }
}

// Client
new VideoPlayerFacade().Play("movie.mp4");
```

## Sources

- https://refactoring.guru/design-patterns/facade

## Related

- [[adapter]]
- [[CS/architecture/proxy]]
- [[mediator]]

## Process

- How does facade differ from adapter in terms of what problem it solves with interfaces?
- Does a facade prevent clients from accessing the subsystem directly, and should it?
- How does facade relate to the concept of a service layer in application architecture?
