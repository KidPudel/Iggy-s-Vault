# Adapter

A structural design pattern that converts the interface of a class into another interface that clients expect.

## What it does

An adapter wraps an incompatible object and translates calls from the target interface into calls on the wrapped object. The client interacts only with the target interface and has no knowledge of the adaptee.

There are two forms: object adapter (composition, wraps an instance) and class adapter (multiple inheritance, less common).

## Code

```csharp
public interface IMediaPlayer { void Play(string file); }

// Incompatible third-party library
public class ThirdPartyAudio { public void PlayAudio(string file, int volume) { } }

// Adapter
public class AudioAdapter : IMediaPlayer {
    private readonly ThirdPartyAudio _lib = new();
    public void Play(string file) => _lib.PlayAudio(file, 100);
}

IMediaPlayer player = new AudioAdapter();
player.Play("song.mp3");
```

## Sources

- https://refactoring.guru/design-patterns/adapter

## Related

- [[facade]]
- [[CS/architecture/proxy]]
- [[decorator]]

## Process

- What is the difference between an object adapter and a class adapter?
- How does adapter differ from facade in terms of interface transformation vs simplification?
- When does wrapping a third-party library in an adapter add value beyond its immediate interface incompatibility?
