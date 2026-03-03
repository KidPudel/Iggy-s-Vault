# Bridge

A structural design pattern that decouples an abstraction from its implementation so both can vary independently.

## What it does

The abstraction holds a reference to an implementation object (the bridge). Instead of inheriting from an implementation, the abstraction delegates to it. New abstractions and new implementations can be added independently without a combinatorial explosion of subclasses.

Useful when both the abstraction and its implementation should be extensible via subclassing.

## Code

```csharp
public interface IRenderer { void RenderCircle(float r); }
public class VectorRenderer : IRenderer { public void RenderCircle(float r) => Console.WriteLine($"Vector circle r={r}"); }
public class RasterRenderer : IRenderer { public void RenderCircle(float r) => Console.WriteLine($"Raster circle r={r}"); }

public abstract class Shape {
    protected IRenderer Renderer;
    protected Shape(IRenderer r) => Renderer = r;
    public abstract void Draw();
}

public class Circle : Shape {
    float _radius;
    public Circle(IRenderer r, float radius) : base(r) => _radius = radius;
    public override void Draw() => Renderer.RenderCircle(_radius);
}

new Circle(new VectorRenderer(), 5).Draw();
new Circle(new RasterRenderer(), 5).Draw();
```

## Sources

- https://refactoring.guru/design-patterns/bridge

## Related

- [[adapter]]
- [[strategy]]
- [[decorator]]

## Process

- How does bridge prevent the "2×N subclass explosion" problem that arises from extending both abstraction and implementation hierarchies?
- What is the structural difference between bridge and strategy?
- Where does the "bridge" physically exist in the code structure?
