# Design Patterns

---

# What Are These Categories?

Design patterns are grouped into three categories based on **what problem they solve**:

## Creational Patterns

**Problem:** How do I create objects?

These patterns deal with object creation mechanisms. Instead of instantiating objects directly with `new`, creational patterns give you flexibility in _what_ gets created, _how_ it gets created, and _when_.

**Think:** "I need to make something, but I want control over how it's made."

## Structural Patterns

**Problem:** How do I compose objects into larger structures?

These patterns deal with relationships between objects—how classes and objects are combined to form larger structures. They help ensure that when parts of a system change, the entire structure doesn't need to change.

**Think:** "I have existing pieces, how do I connect them together?"

## Behavioral Patterns

**Problem:** How do objects communicate and share responsibilities?

These patterns deal with algorithms and the assignment of responsibilities between objects. They describe not just patterns of objects but patterns of communication between them.

**Think:** "I have objects, how should they talk to each other and divide work?"

---

# Creational Patterns

## Singleton

**One instance, global access.**

Ensures a class has only one instance and provides a global point of access to it.

```csharp
public sealed class Logger
{
    private static Logger _instance;
    private static readonly object _lock = new();

    private Logger() { }

    public static Logger Instance
    {
        get
        {
            lock (_lock)
            {
                _instance ??= new Logger();
                return _instance;
            }
        }
    }

    public void Log(string message) => Console.WriteLine($"[LOG] {message}");
}

// Usage
Logger.Instance.Log("Application started");
```

**When to use:** Configuration, logging, connection pools—anything that must have exactly one instance.

**Warning:** Creates global state, makes testing harder. Use sparingly.

---

## Factory Method

**Let subclasses decide what to create.**

Defines an interface for creating objects, but lets subclasses decide which class to instantiate.

```csharp
public abstract class Document
{
    public abstract void Open();
}

public class PdfDocument : Document
{
    public override void Open() => Console.WriteLine("Opening PDF...");
}

public class WordDocument : Document
{
    public override void Open() => Console.WriteLine("Opening Word doc...");
}

public abstract class DocumentCreator
{
    public abstract Document CreateDocument();
    
    public void OpenDocument()
    {
        var doc = CreateDocument();
        doc.Open();
    }
}

public class PdfCreator : DocumentCreator
{
    public override Document CreateDocument() => new PdfDocument();
}

public class WordCreator : DocumentCreator
{
    public override Document CreateDocument() => new WordDocument();
}

// Usage
DocumentCreator creator = new PdfCreator();
creator.OpenDocument();
```

**When to use:** When a class can't anticipate the type of objects it needs to create.

---

## Abstract Factory

**Create families of related objects.**

Provides an interface for creating families of related objects without specifying their concrete classes.

```csharp
// Product interfaces
public interface IButton { void Render(); }
public interface ICheckbox { void Render(); }

// Windows family
public class WindowsButton : IButton
{
    public void Render() => Console.WriteLine("[Windows Button]");
}
public class WindowsCheckbox : ICheckbox
{
    public void Render() => Console.WriteLine("[Windows Checkbox]");
}

// Mac family
public class MacButton : IButton
{
    public void Render() => Console.WriteLine("[Mac Button]");
}
public class MacCheckbox : ICheckbox
{
    public void Render() => Console.WriteLine("[Mac Checkbox]");
}

// Factory interface
public interface IUIFactory
{
    IButton CreateButton();
    ICheckbox CreateCheckbox();
}

public class WindowsFactory : IUIFactory
{
    public IButton CreateButton() => new WindowsButton();
    public ICheckbox CreateCheckbox() => new WindowsCheckbox();
}

public class MacFactory : IUIFactory
{
    public IButton CreateButton() => new MacButton();
    public ICheckbox CreateCheckbox() => new MacCheckbox();
}

// Usage
IUIFactory factory = new MacFactory();
var button = factory.CreateButton();
var checkbox = factory.CreateCheckbox();
button.Render();    // [Mac Button]
checkbox.Render();  // [Mac Checkbox]
```

**When to use:** When your system needs to support multiple product families (themes, platforms, etc.).

---

## Builder

**Construct complex objects step by step.**

Separates the construction of a complex object from its representation.

```csharp
public class Pizza
{
    public string Dough { get; set; }
    public string Sauce { get; set; }
    public List<string> Toppings { get; set; } = new();

    public override string ToString() =>
        $"{Dough} dough, {Sauce} sauce, toppings: {string.Join(", ", Toppings)}";
}

public class PizzaBuilder
{
    private readonly Pizza _pizza = new();

    public PizzaBuilder SetDough(string dough)
    {
        _pizza.Dough = dough;
        return this;
    }

    public PizzaBuilder SetSauce(string sauce)
    {
        _pizza.Sauce = sauce;
        return this;
    }

    public PizzaBuilder AddTopping(string topping)
    {
        _pizza.Toppings.Add(topping);
        return this;
    }

    public Pizza Build() => _pizza;
}

// Usage
var pizza = new PizzaBuilder()
    .SetDough("thin crust")
    .SetSauce("tomato")
    .AddTopping("cheese")
    .AddTopping("pepperoni")
    .Build();

Console.WriteLine(pizza);
// thin crust dough, tomato sauce, toppings: cheese, pepperoni
```

**When to use:** When creating an object requires many steps or configurations.

---

## Prototype

**Clone existing objects.**

Creates new objects by copying an existing object (prototype).

```csharp
public abstract class Shape
{
    public int X { get; set; }
    public int Y { get; set; }
    public string Color { get; set; }

    public abstract Shape Clone();
}

public class Circle : Shape
{
    public int Radius { get; set; }

    public override Shape Clone()
    {
        return new Circle
        {
            X = X,
            Y = Y,
            Color = Color,
            Radius = Radius
        };
    }
}

public class Rectangle : Shape
{
    public int Width { get; set; }
    public int Height { get; set; }

    public override Shape Clone()
    {
        return new Rectangle
        {
            X = X,
            Y = Y,
            Color = Color,
            Width = Width,
            Height = Height
        };
    }
}

// Usage
var original = new Circle { X = 10, Y = 20, Color = "Red", Radius = 5 };
var copy = (Circle)original.Clone();
copy.Color = "Blue";  // Original stays Red
```

**When to use:** When creating an object is expensive, or you need copies with slight variations.

---

# Structural Patterns

## Adapter

**Make incompatible interfaces work together.**

Converts the interface of a class into another interface clients expect.

```csharp
// Existing interface your code uses
public interface IMediaPlayer
{
    void Play(string filename);
}

// Third-party library with different interface
public class ThirdPartyAudioLib
{
    public void PlayAudio(string file, int volume) =>
        Console.WriteLine($"Playing {file} at volume {volume}");
}

// Adapter makes them compatible
public class AudioAdapter : IMediaPlayer
{
    private readonly ThirdPartyAudioLib _audioLib = new();

    public void Play(string filename)
    {
        _audioLib.PlayAudio(filename, 100);
    }
}

// Usage
IMediaPlayer player = new AudioAdapter();
player.Play("song.mp3");
```

**When to use:** When you want to use an existing class but its interface doesn't match what you need.

---

## Composite

**Treat individual objects and compositions uniformly.**

Composes objects into tree structures and lets clients treat individual objects and compositions the same way.

```csharp
public abstract class FileSystemItem
{
    public string Name { get; set; }
    public abstract long GetSize();
}

public class File : FileSystemItem
{
    public long Size { get; set; }
    public override long GetSize() => Size;
}

public class Folder : FileSystemItem
{
    private readonly List<FileSystemItem> _items = new();

    public void Add(FileSystemItem item) => _items.Add(item);
    public void Remove(FileSystemItem item) => _items.Remove(item);

    public override long GetSize() => _items.Sum(item => item.GetSize());
}

// Usage
var root = new Folder { Name = "root" };
var docs = new Folder { Name = "docs" };
docs.Add(new File { Name = "resume.pdf", Size = 1000 });
docs.Add(new File { Name = "photo.jpg", Size = 2000 });
root.Add(docs);
root.Add(new File { Name = "readme.txt", Size = 100 });

Console.WriteLine(root.GetSize());  // 3100
```

**When to use:** When you have tree structures (file systems, UI hierarchies, organization charts).

---

## Proxy

**Control access to an object.**

Provides a surrogate or placeholder for another object to control access to it.

```csharp
public interface IImage
{
    void Display();
}

// Heavy object - expensive to create
public class RealImage : IImage
{
    private readonly string _filename;

    public RealImage(string filename)
    {
        _filename = filename;
        LoadFromDisk();
    }

    private void LoadFromDisk() =>
        Console.WriteLine($"Loading {_filename} from disk...");

    public void Display() =>
        Console.WriteLine($"Displaying {_filename}");
}

// Proxy - delays creation until needed
public class ImageProxy : IImage
{
    private readonly string _filename;
    private RealImage _realImage;

    public ImageProxy(string filename) => _filename = filename;

    public void Display()
    {
        _realImage ??= new RealImage(_filename);
        _realImage.Display();
    }
}

// Usage
IImage image = new ImageProxy("photo.jpg");
// Image not loaded yet...
image.Display();  // Now it loads and displays
image.Display();  // Uses cached image
```

**Types:** Lazy loading (virtual proxy), access control (protection proxy), logging (logging proxy), caching.

**When to use:** When you need to control access, add lazy loading, or add functionality without changing the original class.

---

## Flyweight

**Share common state between objects.**

Minimizes memory use by sharing as much data as possible with similar objects.

```csharp
// Shared state (intrinsic) - stored in flyweight
public class TreeType
{
    public string Name { get; }
    public string Color { get; }
    public string Texture { get; }

    public TreeType(string name, string color, string texture)
    {
        Name = name;
        Color = color;
        Texture = texture;
    }

    public void Draw(int x, int y) =>
        Console.WriteLine($"Drawing {Name} tree at ({x}, {y})");
}

// Flyweight factory
public class TreeFactory
{
    private static readonly Dictionary<string, TreeType> _types = new();

    public static TreeType GetTreeType(string name, string color, string texture)
    {
        var key = $"{name}_{color}_{texture}";
        if (!_types.ContainsKey(key))
        {
            _types[key] = new TreeType(name, color, texture);
        }
        return _types[key];
    }
}

// Unique state (extrinsic) - stored outside
public class Tree
{
    public int X { get; }
    public int Y { get; }
    public TreeType Type { get; }

    public Tree(int x, int y, TreeType type)
    {
        X = x;
        Y = y;
        Type = type;
    }

    public void Draw() => Type.Draw(X, Y);
}

// Usage - 1000 trees but only a few TreeType objects
var forest = new List<Tree>();
var oakType = TreeFactory.GetTreeType("Oak", "Green", "oak_texture.png");
for (int i = 0; i < 1000; i++)
{
    forest.Add(new Tree(Random.Shared.Next(100), Random.Shared.Next(100), oakType));
}
```

**When to use:** When you have many similar objects and memory is a concern.

---

## Facade

**Simple interface to a complex system.**

Provides a unified interface to a set of interfaces in a subsystem.

```csharp
// Complex subsystem classes
public class VideoDecoder
{
    public void Decode(string file) => Console.WriteLine($"Decoding {file}");
}

public class AudioDecoder
{
    public void Decode(string file) => Console.WriteLine($"Decoding audio from {file}");
}

public class VideoBuffer
{
    public void Buffer() => Console.WriteLine("Buffering video...");
}

public class Display
{
    public void Render() => Console.WriteLine("Rendering to screen...");
}

// Facade - simple interface
public class VideoPlayerFacade
{
    private readonly VideoDecoder _videoDecoder = new();
    private readonly AudioDecoder _audioDecoder = new();
    private readonly VideoBuffer _buffer = new();
    private readonly Display _display = new();

    public void Play(string filename)
    {
        _videoDecoder.Decode(filename);
        _audioDecoder.Decode(filename);
        _buffer.Buffer();
        _display.Render();
    }
}

// Usage - client doesn't need to know the complexity
var player = new VideoPlayerFacade();
player.Play("movie.mp4");
```

**When to use:** When you want to provide a simple interface to a complex subsystem.

---

## Bridge

**Separate abstraction from implementation.**

Decouples an abstraction from its implementation so both can vary independently.

```csharp
// Implementation hierarchy
public interface IRenderer
{
    void RenderCircle(float radius);
}

public class VectorRenderer : IRenderer
{
    public void RenderCircle(float radius) =>
        Console.WriteLine($"Drawing circle with radius {radius} as vectors");
}

public class RasterRenderer : IRenderer
{
    public void RenderCircle(float radius) =>
        Console.WriteLine($"Drawing circle with radius {radius} as pixels");
}

// Abstraction hierarchy
public abstract class Shape
{
    protected IRenderer Renderer;

    protected Shape(IRenderer renderer) => Renderer = renderer;

    public abstract void Draw();
}

public class Circle : Shape
{
    private readonly float _radius;

    public Circle(IRenderer renderer, float radius) : base(renderer)
    {
        _radius = radius;
    }

    public override void Draw() => Renderer.RenderCircle(_radius);
}

// Usage - any shape with any renderer
var vectorCircle = new Circle(new VectorRenderer(), 5);
var rasterCircle = new Circle(new RasterRenderer(), 5);
vectorCircle.Draw();  // Drawing circle with radius 5 as vectors
rasterCircle.Draw();  // Drawing circle with radius 5 as pixels
```

**When to use:** When you want to avoid a permanent binding between abstraction and implementation, or both need to be extensible.

---

## Decorator

**Add responsibilities dynamically.**

Attaches additional responsibilities to an object dynamically without subclassing.

```csharp
public interface ICoffee
{
    string GetDescription();
    decimal GetCost();
}

public class SimpleCoffee : ICoffee
{
    public string GetDescription() => "Coffee";
    public decimal GetCost() => 2.00m;
}

public abstract class CoffeeDecorator : ICoffee
{
    protected readonly ICoffee _coffee;
    protected CoffeeDecorator(ICoffee coffee) => _coffee = coffee;
    
    public virtual string GetDescription() => _coffee.GetDescription();
    public virtual decimal GetCost() => _coffee.GetCost();
}

public class MilkDecorator : CoffeeDecorator
{
    public MilkDecorator(ICoffee coffee) : base(coffee) { }
    
    public override string GetDescription() => $"{_coffee.GetDescription()}, Milk";
    public override decimal GetCost() => _coffee.GetCost() + 0.50m;
}

public class SugarDecorator : CoffeeDecorator
{
    public SugarDecorator(ICoffee coffee) : base(coffee) { }
    
    public override string GetDescription() => $"{_coffee.GetDescription()}, Sugar";
    public override decimal GetCost() => _coffee.GetCost() + 0.25m;
}

// Usage - stack decorators
ICoffee coffee = new SimpleCoffee();
coffee = new MilkDecorator(coffee);
coffee = new SugarDecorator(coffee);

Console.WriteLine(coffee.GetDescription());  // Coffee, Milk, Sugar
Console.WriteLine(coffee.GetCost());         // 2.75
```

**When to use:** When you need to add behaviors to objects without affecting other objects of the same class.

---

# Behavioral Patterns

## Template Method

**Define algorithm skeleton, let subclasses fill in steps.**

Defines the skeleton of an algorithm, deferring some steps to subclasses.

```csharp
public abstract class DataProcessor
{
    // Template method - defines the algorithm
    public void Process()
    {
        ReadData();
        ProcessData();
        SaveData();
    }

    protected abstract void ReadData();
    protected abstract void ProcessData();
    
    // Hook - optional override
    protected virtual void SaveData() =>
        Console.WriteLine("Saving to default location...");
}

public class CsvProcessor : DataProcessor
{
    protected override void ReadData() =>
        Console.WriteLine("Reading CSV file...");

    protected override void ProcessData() =>
        Console.WriteLine("Parsing CSV rows...");
}

public class JsonProcessor : DataProcessor
{
    protected override void ReadData() =>
        Console.WriteLine("Reading JSON file...");

    protected override void ProcessData() =>
        Console.WriteLine("Parsing JSON objects...");

    protected override void SaveData() =>
        Console.WriteLine("Saving to cloud storage...");
}

// Usage
DataProcessor processor = new CsvProcessor();
processor.Process();
```

**When to use:** When you have an algorithm with invariant parts and parts that vary.

---

## Mediator

**Centralize complex communications.**

Defines an object that encapsulates how a set of objects interact, promoting loose coupling.

```csharp
public interface IChatMediator
{
    void SendMessage(string message, User sender);
    void AddUser(User user);
}

public class ChatRoom : IChatMediator
{
    private readonly List<User> _users = new();

    public void AddUser(User user)
    {
        _users.Add(user);
        user.SetMediator(this);
    }

    public void SendMessage(string message, User sender)
    {
        foreach (var user in _users.Where(u => u != sender))
        {
            user.Receive(message, sender.Name);
        }
    }
}

public class User
{
    public string Name { get; }
    private IChatMediator _mediator;

    public User(string name) => Name = name;

    public void SetMediator(IChatMediator mediator) => _mediator = mediator;

    public void Send(string message) => _mediator.SendMessage(message, this);

    public void Receive(string message, string from) =>
        Console.WriteLine($"{Name} received from {from}: {message}");
}

// Usage
var chatRoom = new ChatRoom();
var alice = new User("Alice");
var bob = new User("Bob");
var charlie = new User("Charlie");

chatRoom.AddUser(alice);
chatRoom.AddUser(bob);
chatRoom.AddUser(charlie);

alice.Send("Hello everyone!");
// Bob received from Alice: Hello everyone!
// Charlie received from Alice: Hello everyone!
```

**When to use:** When objects communicate in complex ways, making dependencies hard to understand.

---

## Chain of Responsibility

**Pass request along a chain of handlers.**

Lets you pass requests along a chain of handlers, where each handler decides to process the request or pass it on.

```csharp
public abstract class SupportHandler
{
    protected SupportHandler _next;

    public SupportHandler SetNext(SupportHandler next)
    {
        _next = next;
        return next;
    }

    public abstract void Handle(string issue, int severity);
}

public class Level1Support : SupportHandler
{
    public override void Handle(string issue, int severity)
    {
        if (severity <= 1)
        {
            Console.WriteLine($"Level 1 handling: {issue}");
        }
        else
        {
            _next?.Handle(issue, severity);
        }
    }
}

public class Level2Support : SupportHandler
{
    public override void Handle(string issue, int severity)
    {
        if (severity <= 2)
        {
            Console.WriteLine($"Level 2 handling: {issue}");
        }
        else
        {
            _next?.Handle(issue, severity);
        }
    }
}

public class Level3Support : SupportHandler
{
    public override void Handle(string issue, int severity)
    {
        Console.WriteLine($"Level 3 (escalated) handling: {issue}");
    }
}

// Usage
var support = new Level1Support();
support
    .SetNext(new Level2Support())
    .SetNext(new Level3Support());

support.Handle("Password reset", 1);    // Level 1 handling
support.Handle("Software bug", 2);       // Level 2 handling
support.Handle("System down", 3);        // Level 3 handling
```

**When to use:** When more than one object may handle a request, and the handler isn't known in advance.

---

## Observer

**Notify dependents of state changes.**

Defines a one-to-many dependency so that when one object changes state, all its dependents are notified.

```csharp
public class Stock
{
    private decimal _price;
    public string Symbol { get; }
    
    public event Action<string, decimal> OnPriceChanged;

    public Stock(string symbol, decimal price)
    {
        Symbol = symbol;
        _price = price;
    }

    public decimal Price
    {
        get => _price;
        set
        {
            if (_price != value)
            {
                _price = value;
                OnPriceChanged?.Invoke(Symbol, _price);
            }
        }
    }
}

public class StockDisplay
{
    public void Update(string symbol, decimal price) =>
        Console.WriteLine($"Display: {symbol} is now ${price}");
}

public class StockAlert
{
    private readonly decimal _threshold;

    public StockAlert(decimal threshold) => _threshold = threshold;

    public void Update(string symbol, decimal price)
    {
        if (price > _threshold)
            Console.WriteLine($"ALERT: {symbol} exceeded ${_threshold}!");
    }
}

// Usage
var stock = new Stock("AAPL", 150);
var display = new StockDisplay();
var alert = new StockAlert(160);

stock.OnPriceChanged += display.Update;
stock.OnPriceChanged += alert.Update;

stock.Price = 155;  // Display updates
stock.Price = 165;  // Display updates + Alert triggers
```

**When to use:** When changes in one object require changing others, and you don't know how many objects need to change.

---

## Strategy

**Swap algorithms at runtime.**

Defines a family of algorithms, encapsulates each one, and makes them interchangeable.

```csharp
public interface ICompressionStrategy
{
    byte[] Compress(byte[] data);
}

public class ZipCompression : ICompressionStrategy
{
    public byte[] Compress(byte[] data)
    {
        Console.WriteLine("Compressing using ZIP...");
        return data; // Simplified
    }
}

public class RarCompression : ICompressionStrategy
{
    public byte[] Compress(byte[] data)
    {
        Console.WriteLine("Compressing using RAR...");
        return data;
    }
}

public class FileCompressor
{
    private ICompressionStrategy _strategy;

    public void SetStrategy(ICompressionStrategy strategy) =>
        _strategy = strategy;

    public byte[] CompressFile(byte[] fileData) =>
        _strategy.Compress(fileData);
}

// Usage
var compressor = new FileCompressor();
var data = new byte[] { 1, 2, 3 };

compressor.SetStrategy(new ZipCompression());
compressor.CompressFile(data);  // Compressing using ZIP...

compressor.SetStrategy(new RarCompression());
compressor.CompressFile(data);  // Compressing using RAR...
```

**When to use:** When you have multiple algorithms for a task and want to switch between them.

---

## Command

**Encapsulate requests as objects.**

Turns a request into a stand-alone object containing all information about the request.

```csharp
public interface ICommand
{
    void Execute();
    void Undo();
}

public class Light
{
    public void On() => Console.WriteLine("Light is ON");
    public void Off() => Console.WriteLine("Light is OFF");
}

public class LightOnCommand : ICommand
{
    private readonly Light _light;
    public LightOnCommand(Light light) => _light = light;
    
    public void Execute() => _light.On();
    public void Undo() => _light.Off();
}

public class LightOffCommand : ICommand
{
    private readonly Light _light;
    public LightOffCommand(Light light) => _light = light;
    
    public void Execute() => _light.Off();
    public void Undo() => _light.On();
}

public class RemoteControl
{
    private readonly Stack<ICommand> _history = new();

    public void Press(ICommand command)
    {
        command.Execute();
        _history.Push(command);
    }

    public void Undo()
    {
        if (_history.Count > 0)
        {
            _history.Pop().Undo();
        }
    }
}

// Usage
var light = new Light();
var remote = new RemoteControl();

remote.Press(new LightOnCommand(light));  // Light is ON
remote.Press(new LightOffCommand(light)); // Light is OFF
remote.Undo();                             // Light is ON
```

**When to use:** When you need to parameterize, queue, log, or undo operations.

---

## State

**Object behavior changes with internal state.**

Allows an object to alter its behavior when its internal state changes.

```csharp
public interface IOrderState
{
    void Next(Order order);
    void Prev(Order order);
    void PrintStatus();
}

public class PendingState : IOrderState
{
    public void Next(Order order) => order.SetState(new ShippedState());
    public void Prev(Order order) => Console.WriteLine("Already at initial state");
    public void PrintStatus() => Console.WriteLine("Order is PENDING");
}

public class ShippedState : IOrderState
{
    public void Next(Order order) => order.SetState(new DeliveredState());
    public void Prev(Order order) => order.SetState(new PendingState());
    public void PrintStatus() => Console.WriteLine("Order is SHIPPED");
}

public class DeliveredState : IOrderState
{
    public void Next(Order order) => Console.WriteLine("Order already delivered");
    public void Prev(Order order) => order.SetState(new ShippedState());
    public void PrintStatus() => Console.WriteLine("Order is DELIVERED");
}

public class Order
{
    private IOrderState _state = new PendingState();

    public void SetState(IOrderState state) => _state = state;
    public void NextState() => _state.Next(this);
    public void PrevState() => _state.Prev(this);
    public void PrintStatus() => _state.PrintStatus();
}

// Usage
var order = new Order();
order.PrintStatus();  // PENDING
order.NextState();
order.PrintStatus();  // SHIPPED
order.NextState();
order.PrintStatus();  // DELIVERED
```

**When to use:** When an object's behavior depends on its state, and it must change behavior at runtime.

---

## Visitor

**Add operations without changing classes.**

Lets you define a new operation without changing the classes of the elements on which it operates.

```csharp
public interface IShapeVisitor
{
    void Visit(Circle circle);
    void Visit(Rectangle rectangle);
}

public interface IShape
{
    void Accept(IShapeVisitor visitor);
}

public class Circle : IShape
{
    public double Radius { get; set; }
    public void Accept(IShapeVisitor visitor) => visitor.Visit(this);
}

public class Rectangle : IShape
{
    public double Width { get; set; }
    public double Height { get; set; }
    public void Accept(IShapeVisitor visitor) => visitor.Visit(this);
}

public class AreaCalculator : IShapeVisitor
{
    public double TotalArea { get; private set; }

    public void Visit(Circle circle) =>
        TotalArea += Math.PI * circle.Radius * circle.Radius;

    public void Visit(Rectangle rectangle) =>
        TotalArea += rectangle.Width * rectangle.Height;
}

public class DrawingVisitor : IShapeVisitor
{
    public void Visit(Circle circle) =>
        Console.WriteLine($"Drawing circle with radius {circle.Radius}");

    public void Visit(Rectangle rectangle) =>
        Console.WriteLine($"Drawing rectangle {rectangle.Width}x{rectangle.Height}");
}

// Usage
var shapes = new List<IShape>
{
    new Circle { Radius = 5 },
    new Rectangle { Width = 4, Height = 6 }
};

var areaCalc = new AreaCalculator();
foreach (var shape in shapes)
{
    shape.Accept(areaCalc);
}
Console.WriteLine($"Total area: {areaCalc.TotalArea}");
```

**When to use:** When you need to perform operations across a structure of different types and want to keep the operations separate from the structure.

---

## Iterator

**Access elements sequentially without exposing structure.**

Provides a way to access elements of a collection sequentially without exposing its underlying representation.

```csharp
public class Book
{
    public string Title { get; set; }
}

public class BookCollection : IEnumerable<Book>
{
    private readonly List<Book> _books = new();

    public void Add(Book book) => _books.Add(book);

    public IEnumerator<Book> GetEnumerator() => _books.GetEnumerator();
    IEnumerator IEnumerable.GetEnumerator() => GetEnumerator();
}

// Usage
var library = new BookCollection();
library.Add(new Book { Title = "Design Patterns" });
library.Add(new Book { Title = "Clean Code" });

foreach (var book in library)
{
    Console.WriteLine(book.Title);
}
```

**Note:** C# has built-in iterator support via `IEnumerable` and `yield return`. Use those.

**When to use:** When you need to traverse a collection without exposing its internal structure.

---

## Interpreter

**Define a grammar and interpret sentences.**

Defines a representation for a language's grammar and provides an interpreter to process sentences in that language.

```csharp
public interface IExpression
{
    int Interpret();
}

public class Number : IExpression
{
    private readonly int _value;
    public Number(int value) => _value = value;
    public int Interpret() => _value;
}

public class Add : IExpression
{
    private readonly IExpression _left, _right;
    public Add(IExpression left, IExpression right)
    {
        _left = left;
        _right = right;
    }
    public int Interpret() => _left.Interpret() + _right.Interpret();
}

public class Subtract : IExpression
{
    private readonly IExpression _left, _right;
    public Subtract(IExpression left, IExpression right)
    {
        _left = left;
        _right = right;
    }
    public int Interpret() => _left.Interpret() - _right.Interpret();
}

// Usage: (5 + 10) - 3
IExpression expression = new Subtract(
    new Add(new Number(5), new Number(10)),
    new Number(3)
);

Console.WriteLine(expression.Interpret());  // 12
```

**When to use:** When you have a simple language to interpret (rules engines, simple DSLs, math expressions).

---

## Memento

**Capture and restore object state.**

Captures an object's internal state so it can be restored later without violating encapsulation.

```csharp
public class EditorMemento
{
    public string Content { get; }
    public EditorMemento(string content) => Content = content;
}

public class TextEditor
{
    public string Content { get; set; } = "";

    public EditorMemento Save() => new EditorMemento(Content);

    public void Restore(EditorMemento memento) => Content = memento.Content;
}

public class History
{
    private readonly Stack<EditorMemento> _states = new();

    public void Push(EditorMemento memento) => _states.Push(memento);

    public EditorMemento Pop() =>
        _states.Count > 0 ? _states.Pop() : null;
}

// Usage
var editor = new TextEditor();
var history = new History();

editor.Content = "Hello";
history.Push(editor.Save());

editor.Content = "Hello World";
history.Push(editor.Save());

editor.Content = "Hello World!!!";

Console.WriteLine(editor.Content);  // Hello World!!!
editor.Restore(history.Pop());
Console.WriteLine(editor.Content);  // Hello World
editor.Restore(history.Pop());
Console.WriteLine(editor.Content);  // Hello
```

**When to use:** When you need undo functionality or snapshots.

---

# Quick Reference

| Pattern                 | Category   | One-Line Summary                             |
| ----------------------- | ---------- | -------------------------------------------- |
| Singleton               | Creational | One instance, global access                  |
| Factory Method          | Creational | Subclasses decide what to create             |
| Abstract Factory        | Creational | Create families of related objects           |
| Builder                 | Creational | Construct complex objects step by step       |
| Prototype               | Creational | Clone existing objects                       |
| Adapter                 | Structural | Make incompatible interfaces work together   |
| Composite               | Structural | Treat individuals and groups uniformly       |
| Proxy                   | Structural | Control access to an object                  |
| Flyweight               | Structural | Share common state to save memory            |
| Facade                  | Structural | Simple interface to complex system           |
| Bridge                  | Structural | Separate abstraction from implementation     |
| Decorator               | Structural | Add responsibilities dynamically             |
| Template Method         | Behavioral | Algorithm skeleton with variable steps       |
| Mediator                | Behavioral | Centralize complex communications            |
| Chain of Responsibility | Behavioral | Pass request along handler chain             |
| Observer                | Behavioral | Notify dependents of state changes           |
| Strategy                | Behavioral | Swap algorithms at runtime                   |
| Command                 | Behavioral | Encapsulate requests as objects              |
| State                   | Behavioral | Behavior changes with state                  |
| Visitor                 | Behavioral | Add operations without changing classes      |
| Iterator                | Behavioral | Sequential access without exposing structure |
| Interpreter             | Behavioral | Define and interpret a grammar               |
| Memento                 | Behavioral | Capture and restore state                    |
|                         |            |                                              |


# When to Use What

## Object Creation Problems

**Need to create objects without specifying exact class?** → Factory Method  
**Need families of related objects?** → Abstract Factory  
**Need to construct complex objects step by step?** → Builder  
**Need copies of existing objects?** → Prototype  
**Need exactly one global instance?** → Singleton _(use sparingly)_

## Structure & Composition Problems

**Need to adapt incompatible interfaces?** → Adapter  
**Need tree/hierarchical structures?** → Composite  
**Need lazy loading or access control?** → Proxy  
**Need memory efficiency with many similar objects?** → Flyweight  
**Need to simplify a complex subsystem?** → Facade  
**Need abstraction and implementation to vary independently?** → Bridge  
**Need to add behavior without subclassing?** → Decorator

## Behavior & Communication Problems

**Need swappable algorithms?** → Strategy  
**Need object behavior to change with its state?** → State  
**Need to notify multiple objects of changes?** → Observer  
**Need undo/redo or replay?** → Command + Memento  
**Need request handled by one of many handlers?** → Chain of Responsibility  
**Need to reduce chaotic dependencies?** → Mediator

## Algorithm & Processing Problems

**Need algorithm skeleton with varying steps?** → Template Method  
**Need to traverse collection without exposing internals?** → Iterator  
**Need to add operations without modifying classes?** → Visitor  
**Need a simple scripting/rules language?** → Interpreter