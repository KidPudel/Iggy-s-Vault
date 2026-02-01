# C# for Polyglot Programmers

> The "why" behind C# features. Not just syntax — when and why to use each.

---

## Value vs Reference Types

The most fundamental thing to understand in C#.

```csharp
// Value type: copied on assignment
int a = 5;
int b = a;      // b is independent copy
b = 10;         // a is still 5

// Reference type: reference copied, same object
List<int> listA = new List<int> { 1, 2 };
List<int> listB = listA;    // Both point to SAME list
listB.Add(3);               // listA also sees the 3
```

**Why this matters:**

- Passing a struct to a method? It's copied. Changes inside don't affect original.
- Passing a class? Same object. Changes persist.
- Returning a list from a method? Caller can modify your internal list.

|Value Types|Reference Types|
|---|---|
|`int, float, double, bool, char`|`class` instances|
|`struct`|`string` (but immutable)|
|`enum`|arrays, `List<T>`, `Dictionary`|

**When to use struct:**

- Small data (under 16 bytes)
- Immutable preferred
- Want copy semantics
- Example: `Vector3`, `Color`, `Quaternion` are all structs

---

## Nullable Types

**Problem:** Value types can't be null. But sometimes "no value" is meaningful.

```csharp
int x = null;       // ERROR: int can't be null

int? x = null;      // OK: int? is "nullable int"
int? y = 5;         // Can also hold a value

// Check before using
if (x.HasValue)
    int val = x.Value;

// Or use null coalescing
int val = x ?? 0;   // Use 0 if x is null

// Or GetValueOrDefault
int val = x.GetValueOrDefault();  // 0 if null
```

**When to use:**

- Database fields that can be NULL
- Optional configuration values
- "Not yet set" states
- Return values where "nothing found" is valid

**Nullable reference types (C# 8+):**

```csharp
#nullable enable

string name = null;     // Warning: non-nullable assigned null
string? name = null;    // OK: explicitly nullable

// Forces you to handle null cases
void Greet(string? name)
{
    if (name is null) return;
    Console.WriteLine(name.Length);  // Safe: compiler knows it's not null here
}
```

**Why this exists:** Eliminates "billion dollar mistake" — null reference exceptions at runtime. Compiler catches them at build time.

---

## var Keyword

**What it does:** Compiler infers the type. Not dynamic — still strongly typed.

```csharp
var x = 10;                     // Compiler knows: int
var name = "hello";             // string
var list = new List<int>();     // List<int>
var enemy = GetEnemy();         // Whatever GetEnemy returns

// Type is locked at compile time
var x = 10;
x = "hello";    // ERROR: can't assign string to int
```

**When to use:**

- When type is obvious from right side: `var dict = new Dictionary<string, List<int>>();`
- LINQ queries where type is complex
- When you'd just repeat yourself: `PlayerController controller = new PlayerController();` → `var controller = new PlayerController();`

**When NOT to use:**

- When type isn't obvious: `var result = Calculate();` — what's the type?
- For primitive assignments where being explicit helps: `int count = 0;`

---

## Strings

### Immutability

```csharp
string s = "hello";
s.ToUpper();        // Returns "HELLO" but s is still "hello"
s = s.ToUpper();    // Must reassign

// Every "modification" creates a new string
string a = "hello";
string b = a + " world";  // New string created, a unchanged
```

**Why immutable:**

- Thread-safe by default
- Can be interned (shared in memory)
- Safe to use as dictionary keys

**Performance trap:**

```csharp
// BAD: Creates thousands of strings
string result = "";
for (int i = 0; i < 1000; i++)
    result += i + ", ";     // New string each iteration

// GOOD: StringBuilder for loops
var sb = new StringBuilder();
for (int i = 0; i < 1000; i++)
    sb.Append(i).Append(", ");
string result = sb.ToString();
```

### String Interpolation

```csharp
// Old way (avoid)
string msg = "Player " + name + " has " + score + " points";

// String.Format (avoid)
string msg = string.Format("Player {0} has {1} points", name, score);

// Interpolation (use this)
string msg = $"Player {name} has {score} points";
string msg = $"Health: {health:F1}%";      // Format: 1 decimal place
string msg = $"Gold: {gold:N0}";           // Format: thousands separator
string msg = $"Damage: {damage,10}";       // Padding: right-align in 10 chars
```

### Verbatim Strings

```csharp
// Normal: must escape backslashes
string path = "C:\\Users\\Name\\file.txt";

// Verbatim: backslashes as-is
string path = @"C:\Users\Name\file.txt";

// Also allows multiline
string json = @"{
    ""name"": ""value""
}";

// Combined with interpolation
string path = $@"C:\Users\{username}\file.txt";
```

### Raw String Literals (C# 11)

```csharp
// Triple quotes: no escaping needed
string json = """
    {
        "name": "value",
        "count": 42
    }
    """;
```

---

## Collections

### Array

Fixed size. Fast. Use when size is known and won't change.

```csharp
int[] arr = new int[5];                // Fixed size, default values (0)
int[] arr = { 1, 2, 3, 4, 5 };         // Initializer
int[] arr = new int[] { 1, 2, 3 };     // Explicit

arr[0] = 10;
int len = arr.Length;       // Property, not method

// Can't resize. This creates a NEW array:
Array.Resize(ref arr, 10);
```

**When to use:** Known fixed size, performance-critical code, interop with APIs expecting arrays.

### `List<T>`

Dynamic size. The go-to collection.

```csharp
var list = new List<int>();
var list = new List<int> { 1, 2, 3 };
var list = new List<int>(capacity: 100);  // Hint at size to avoid resizing

list.Add(4);
list.Remove(3);         // Removes first occurrence of value 3
list.RemoveAt(0);       // Removes at index
list.Insert(0, 99);     // Insert at index
list.Clear();

list.Count              // Current size
list[0]                 // Index access
list.Contains(5)        // O(n) search
list.IndexOf(5)         // -1 if not found
```

**When to use:** Unknown/changing size, need add/remove, most situations.

**Gotcha — removing while iterating:**

```csharp
// WRONG: Modifying collection while iterating
foreach (var item in list)
    if (item > 5) list.Remove(item);    // Throws exception

// RIGHT: Use RemoveAll
list.RemoveAll(item => item > 5);

// RIGHT: Iterate backwards
for (int i = list.Count - 1; i >= 0; i--)
    if (list[i] > 5) list.RemoveAt(i);
```

### `Dictionary<K, V>`

Key-value pairs. O(1) lookup by key.

```csharp
var dict = new Dictionary<string, int>();
var dict = new Dictionary<string, int>
{
    { "health", 100 },
    { "mana", 50 },
    ["stamina"] = 75      // Alternative syntax
};

dict["health"] = 80;      // Add or update
dict.Add("armor", 25);    // Add only — throws if key exists

// WRONG: Throws if key missing
int val = dict["missing"];

// RIGHT: Safe access
if (dict.TryGetValue("health", out int val))
{
    // Use val
}

// Or with default
int val = dict.GetValueOrDefault("health", 0);

dict.ContainsKey("health")
dict.Remove("health");
```

**When to use:** Need fast lookup by key, associating data, caching.

**Gotcha — key requirements:**

- Keys must implement `GetHashCode()` and `Equals()` properly
- Mutable keys = disaster. If key changes after insertion, can't find it.
- Use immutable keys: `string`, `int`, `enum`, immutable structs

### `HashSet<T>`

Unique values only. O(1) contains check.

```csharp
var set = new HashSet<int> { 1, 2, 3 };

set.Add(4);         // Returns true
set.Add(2);         // Returns false (already exists), no error
set.Contains(3);    // O(1) lookup — fast!
set.Remove(2);

// Set operations
set.UnionWith(other);       // Add all from other
set.IntersectWith(other);   // Keep only common
set.ExceptWith(other);      // Remove all in other
```

**When to use:**

- Need unique items
- Need fast "contains" checks
- Tracking visited states
- Removing duplicates: `var unique = new HashSet<int>(listWithDupes);`

### `Queue<T> and Stack<T>`

```csharp
// Queue: FIFO (first in, first out)
var queue = new Queue<string>();
queue.Enqueue("first");
queue.Enqueue("second");
string next = queue.Dequeue();    // "first"
string peek = queue.Peek();       // Look without removing

// Stack: LIFO (last in, first out)  
var stack = new Stack<string>();
stack.Push("first");
stack.Push("second");
string top = stack.Pop();         // "second"
string peek = stack.Peek();
```

**When to use:**

- Queue: Task processing, BFS, event queues, turn order
- Stack: Undo systems, DFS, parsing, call tracking

---

## LINQ

Language-Integrated Query. Transform and query collections declaratively.

### Why LINQ Matters

```csharp
// Without LINQ
List<Enemy> aliveEnemies = new List<Enemy>();
foreach (var enemy in enemies)
{
    if (enemy.Health > 0)
        aliveEnemies.Add(enemy);
}
aliveEnemies.Sort((a, b) => a.Health.CompareTo(b.Health));
Enemy weakest = aliveEnemies.Count > 0 ? aliveEnemies[0] : null;

// With LINQ
Enemy weakest = enemies
    .Where(e => e.Health > 0)
    .OrderBy(e => e.Health)
    .FirstOrDefault();
```

### Core Operations

```csharp
// Filtering — keep items matching condition
enemies.Where(e => e.Health > 0)

// Projection — transform each item
enemies.Select(e => e.Name)                    // List of names
enemies.Select(e => new { e.Name, e.Health })  // Anonymous objects

// First / Last / Single
enemies.First()                     // First item (throws if empty)
enemies.FirstOrDefault()            // First or null/default
enemies.First(e => e.IsBoss)        // First matching
enemies.Last()                      // Last item
enemies.Single()                    // Only item (throws if != 1)
enemies.SingleOrDefault()           // Only item or null

// Aggregation
enemies.Count()
enemies.Count(e => e.IsBoss)        // Count matching
enemies.Sum(e => e.Health)
enemies.Average(e => e.Health)
enemies.Min(e => e.Health)
enemies.Max(e => e.Health)

// Checking
enemies.Any()                       // Not empty?
enemies.Any(e => e.IsBoss)          // Any match?
enemies.All(e => e.Health > 0)      // All match?

// Ordering
enemies.OrderBy(e => e.Health)              // Ascending
enemies.OrderByDescending(e => e.Health)    // Descending
enemies.OrderBy(e => e.Type).ThenBy(e => e.Name)  // Multiple criteria

// Grouping
enemies.GroupBy(e => e.Type)        // IEnumerable<IGrouping<Type, Enemy>>

// Distinct
names.Distinct()

// Take / Skip (pagination)
enemies.Take(10)                    // First 10
enemies.Skip(20).Take(10)           // Items 21-30
enemies.TakeLast(5)                 // Last 5

// Conversion
enemies.ToList()
enemies.ToArray()
enemies.ToDictionary(e => e.Id)
enemies.ToDictionary(e => e.Id, e => e.Name)
enemies.ToHashSet()
```

### LINQ is Lazy

```csharp
// This does NOT execute yet
var query = enemies.Where(e => e.Health > 0);

// NOW it executes (when you enumerate or convert)
var list = query.ToList();
foreach (var e in query) { }

// Each enumeration re-executes the query
foreach (var e in query) { }   // Filters again
foreach (var e in query) { }   // Filters again

// Cache if reusing
var cached = query.ToList();   // Execute once, reuse list
```

**Why lazy?** Allows chaining without intermediate collections. Only computes what's needed.

### Method Chaining

```csharp
var result = enemies
    .Where(e => e.Health > 0)           // Filter alive
    .Where(e => e.Type == EnemyType.Orc) // Filter orcs
    .OrderBy(e => e.Health)              // Sort by health
    .Take(5)                             // Top 5 weakest
    .Select(e => e.Name)                 // Get names only
    .ToList();                           // Execute and materialize
```

---

## Properties

Properties look like fields but have getter/setter logic.

### Why Properties Over Public Fields

```csharp
// Public field — NO control
public int health;
enemy.health = -999;    // Oops, invalid state

// Property — you control access
private int _health;
public int Health
{
    get => _health;
    set => _health = Mathf.Clamp(value, 0, _maxHealth);
}
```

**Benefits:**

- Validation in setter
- Computed values in getter
- Can change implementation without breaking callers
- Can add logic later without API change
- Can have different access levels (public get, private set)

### Property Variants

```csharp
// Auto-implemented (most common)
public int Health { get; set; }

// With default value
public int Health { get; set; } = 100;

// Read-only (set only in constructor)
public int MaxHealth { get; }

// Private setter (external read-only)
public int Health { get; private set; }

// Expression body (computed, read-only)
public bool IsDead => Health <= 0;
public string FullName => $"{FirstName} {LastName}";
public float HealthPercent => (float)Health / MaxHealth;

// Init-only (C# 9+) — set only during initialization
public int Id { get; init; }
var enemy = new Enemy { Id = 5 };   // OK
enemy.Id = 10;                       // ERROR after construction

// Full property with backing field
private int _health;
public int Health
{
    get => _health;
    set
    {
        if (value < 0) value = 0;
        if (_health != value)
        {
            _health = value;
            OnHealthChanged?.Invoke(_health);
        }
    }
}
```

### Unity Serialization with Properties

```csharp
// This WON'T show in Inspector (properties aren't serialized)
public int Health { get; set; }

// This WILL show in Inspector
[field: SerializeField]
public int Health { get; private set; }

// Or use backing field explicitly
[SerializeField] private int _health;
public int Health => _health;
```

---

## Classes vs Structs

### Class (Reference Type)

```csharp
public class Enemy
{
    public int Health { get; set; }
}

Enemy a = new Enemy { Health = 100 };
Enemy b = a;            // Same object
b.Health = 50;          // a.Health is also 50
```

- Heap allocated
- Reference copied on assignment
- Can be null
- Supports inheritance
- Default: suitable for most game objects

### Struct (Value Type)

```csharp
public struct DamageInfo
{
    public int Amount;
    public DamageType Type;
    public Vector3 HitPoint;
}

DamageInfo a = new DamageInfo { Amount = 10 };
DamageInfo b = a;       // Copy created
b.Amount = 20;          // a.Amount is still 10
```

- Stack allocated (usually)
- Copied on assignment
- Cannot be null (unless nullable)
- No inheritance
- Best for small, immutable data

### When to Use Which

|Use Struct|Use Class|
|---|---|
|Small (< 16 bytes)|Larger objects|
|Immutable data|Mutable state|
|Short-lived|Long-lived|
|No inheritance needed|Need inheritance|
|Want copy semantics|Want reference semantics|
|Examples: Vector3, Color, Ray|Examples: Enemy, Player, GameManager|

**Gotcha — struct in collection:**

```csharp
struct Enemy { public int Health; }

List<Enemy> enemies = new List<Enemy> { new Enemy { Health = 100 } };
enemies[0].Health = 50;   // ERROR: can't modify struct returned by indexer

// The indexer returns a COPY, not a reference
// Must replace entirely:
var enemy = enemies[0];
enemy.Health = 50;
enemies[0] = enemy;
```

---

## Access Modifiers and Inheritance Keywords

### Access Modifiers

Control who can see and use your code.

```csharp
public class Example
{
    public int PublicField;           // Anyone can access
    private int _privateField;        // Only this class
    protected int ProtectedField;     // This class + derived classes
    internal int InternalField;       // Same assembly only
    protected internal int ProtectedInternalField;  // Same assembly OR derived classes
    private protected int PrivateProtectedField;    // Derived classes in same assembly only
}
```

|Modifier|Same Class|Derived (Same Assembly)|Derived (Other Assembly)|Same Assembly|Other Assembly|
|---|---|---|---|---|---|
|`public`|✓|✓|✓|✓|✓|
|`private`|✓|✗|✗|✗|✗|
|`protected`|✓|✓|✓|✗|✗|
|`internal`|✓|✓|✗|✓|✗|
|`protected internal`|✓|✓|✓|✓|✗|
|`private protected`|✓|✓|✗|✗|✗|

**When to use:**

- `public` — API that others should use
- `private` — Implementation details (default for fields)
- `protected` — Intended for subclasses to use/override
- `internal` — Share within your assembly, hide from external code
- `protected internal` — Rare; library extension points
- `private protected` — Rare; restrict inheritance to same assembly

```csharp
public class Player : MonoBehaviour
{
    // Public API
    public int Health { get; private set; }
    public void TakeDamage(int amount) { }
    
    // Internal implementation
    private float _invincibilityTimer;
    private void UpdateInvincibility() { }
    
    // For subclasses
    protected virtual void OnDeath() { }
}
```

### virtual — Allow Override

Marks a method/property as overridable by derived classes.

```csharp
public class Enemy
{
    public virtual void Attack()
    {
        Debug.Log("Basic attack");
    }
    
    public virtual int Damage => 10;
}

public class Boss : Enemy
{
    public override void Attack()
    {
        Debug.Log("Powerful attack!");
    }
    
    public override int Damage => 50;
}
```

**Without `virtual`:**

```csharp
public class Enemy
{
    public void Attack() { }  // NOT virtual
}

public class Boss : Enemy
{
    public void Attack() { }  // This HIDES, doesn't override
}

Enemy enemy = new Boss();
enemy.Attack();  // Calls Enemy.Attack(), not Boss.Attack()!
```

### override — Replace Base Implementation

Must be used when overriding a `virtual` or `abstract` member.

```csharp
public class Orc : Enemy
{
    public override void Attack()
    {
        // Can call base implementation
        base.Attack();
        
        // Then add behavior
        PlayRoarSound();
    }
}
```

**Common mistake:**

```csharp
// WRONG — forgot 'override'
public class Orc : Enemy
{
    public void Attack() { }  // Warning: hides inherited member
}

// RIGHT
public class Orc : Enemy
{
    public override void Attack() { }
}
```

### abstract — Must Be Implemented

Cannot be instantiated. Forces derived classes to implement.

```csharp
// Abstract class — can't do: new Enemy()
public abstract class Enemy
{
    public int Health { get; protected set; }
    
    // Abstract method — no body, derived MUST implement
    public abstract void Attack();
    
    // Abstract property
    public abstract int Damage { get; }
    
    // Can still have normal methods
    public void TakeDamage(int amount)
    {
        Health -= amount;
    }
}

public class Goblin : Enemy
{
    // MUST implement abstract members
    public override void Attack()
    {
        Debug.Log("Goblin stab!");
    }
    
    public override int Damage => 5;
}
```

**When to use:**

- Base class that doesn't make sense on its own
- Force all subclasses to implement specific behavior
- Template method pattern

**abstract vs interface:**

- Abstract class: Can have implementation, fields, one inheritance only
- Interface: No implementation (mostly), multiple inheritance allowed

### sealed — Prevent Inheritance

Stops further inheritance or overriding.

```csharp
// Sealed class — cannot be inherited
public sealed class FinalBoss : Enemy
{
    // No one can derive from FinalBoss
}

// Sealed method — stops override chain
public class Boss : Enemy
{
    public sealed override void Attack()
    {
        // Derived classes can't override Attack anymore
    }
}
```

**Why seal?**

- Security: Prevent someone overriding your validation logic
- Performance: Compiler can optimize sealed calls
- Design: Signal "this is final, don't extend"

```csharp
// Common pattern: seal leaf classes in hierarchies
public abstract class Enemy { }
public class Orc : Enemy { }           // Could seal
public sealed class OrcChieftain : Orc { }  // Can't extend further
```

### new — Hide Inherited Member

Explicitly hide a base class member (usually a warning sign).

```csharp
public class Base
{
    public void DoSomething() => Debug.Log("Base");
}

public class Derived : Base
{
    // Hides Base.DoSomething (NOT polymorphic)
    public new void DoSomething() => Debug.Log("Derived");
}

Derived d = new Derived();
d.DoSomething();           // "Derived"

Base b = d;
b.DoSomething();           // "Base" — NOT "Derived"!
```

**When to use `new`:**

- Almost never intentionally
- Usually indicates design problem
- Legitimate use: Returning more specific type

```csharp
public class AnimalShelter
{
    public virtual Animal GetPet() => new Animal();
}

public class CatShelter : AnimalShelter
{
    // Return more specific type
    public new Cat GetPet() => new Cat();
}
```

### static — Belongs to Type, Not Instance

```csharp
public class Enemy
{
    // Instance member — each enemy has its own
    public int Health { get; set; }
    
    // Static member — shared by all enemies
    public static int TotalKilled { get; private set; }
    
    // Static method
    public static void ResetKillCount() => TotalKilled = 0;
    
    public void Die()
    {
        TotalKilled++;  // Increment shared counter
    }
}

// Access via type name
Enemy.TotalKilled
Enemy.ResetKillCount();
```

**Static class:**

```csharp
public static class GameUtils
{
    public static float Remap(float value, float from1, float to1, float from2, float to2)
    {
        return (value - from1) / (to1 - from1) * (to2 - from2) + from2;
    }
}
```

### const vs readonly vs static readonly

```csharp
public class Config
{
    // const: Compile-time constant. Inlined everywhere. Can't be changed.
    public const int MaxPlayers = 4;
    public const string Version = "1.0.0";
    
    // readonly: Set at declaration or in constructor only
    public readonly int Seed;
    public Config(int seed) { Seed = seed; }
    
    // static readonly: Like const but computed at runtime
    public static readonly DateTime StartTime = DateTime.Now;
    public static readonly int ProcessorCount = Environment.ProcessorCount;
}
```

||`const`|`readonly`|`static readonly`|
|---|---|---|---|
|When set|Compile time|Declaration/constructor|Declaration/static constructor|
|Can be instance|No (implicit static)|Yes|No|
|Can use runtime values|No|Yes|Yes|
|Inlined at compile time|Yes|No|No|

**Use `const` for:** True constants that never change (math values, fixed strings) **Use `readonly` for:** Values set once per instance **Use `static readonly` for:** Runtime-computed values shared across instances

### partial — Split Across Files

Split a class/struct/interface across multiple files.

```csharp
// Player.cs
public partial class Player
{
    public int Health { get; set; }
    public void TakeDamage(int amount) { }
}

// Player.Movement.cs
public partial class Player
{
    public float Speed { get; set; }
    public void Move(Vector3 direction) { }
}

// Player.Combat.cs
public partial class Player
{
    public void Attack() { }
    public void Block() { }
}
```

**When to use:**

- Large classes organized by concern
- Code generation (Unity serialization, protobuf)
- Separate auto-generated code from manual code

### Inheritance Keywords Summary

|Keyword|On Class|On Member|Purpose|
|---|---|---|---|
|`virtual`|-|✓|Allow derived classes to override|
|`override`|-|✓|Replace base implementation|
|`abstract`|✓|✓|Force derived classes to implement|
|`sealed`|✓|✓|Prevent inheritance/override|
|`new`|-|✓|Hide (not override) base member|
|`static`|✓|✓|Belongs to type, not instance|
|`partial`|✓|✓|Split across files|

### Common Patterns

**Template Method:**

```csharp
public abstract class GameMode
{
    // Template method — defines the algorithm
    public void StartGame()
    {
        InitializePlayers();
        SpawnEnemies();
        StartTimer();
    }
    
    protected abstract void InitializePlayers();
    protected abstract void SpawnEnemies();
    protected virtual void StartTimer() { /* default implementation */ }
}

public class SurvivalMode : GameMode
{
    protected override void InitializePlayers() { /* ... */ }
    protected override void SpawnEnemies() { /* endless waves */ }
    // Uses default StartTimer
}
```

**Protected virtual for Unity:**

```csharp
public class Character : MonoBehaviour
{
    protected virtual void Awake() { /* base setup */ }
    protected virtual void Start() { /* base init */ }
    protected virtual void Update() { /* base loop */ }
}

public class Player : Character
{
    protected override void Awake()
    {
        base.Awake();  // Don't forget base!
        // Player-specific setup
    }
}
```

---

## Records (C# 9+)

Immutable reference types with value equality. Perfect for data transfer objects.

```csharp
// One line defines a complete immutable class
public record Person(string Name, int Age);

// Automatically gets:
// - Constructor
// - Properties (init-only)
// - Value equality (not reference equality)
// - ToString()
// - Deconstruction

var p1 = new Person("Alice", 30);
var p2 = new Person("Alice", 30);

Console.WriteLine(p1 == p2);    // true (value equality!)
Console.WriteLine(p1);          // "Person { Name = Alice, Age = 30 }"

// Non-destructive mutation with 'with'
var p3 = p1 with { Age = 31 };  // New object, Alice is now 31

// Deconstruction
var (name, age) = p1;
```

**When to use:**

- DTOs (Data Transfer Objects)
- Immutable configuration
- Event data
- Anything where two instances with same values should be "equal"

**Record struct (C# 10):**

```csharp
public readonly record struct Point(float X, float Y);
// Value type + value equality + immutability
```

---

## Interfaces

Contracts that define what a type can do, not how it does it.

### Why Interfaces

```csharp
// Without interface — coupled to specific type
void Attack(Enemy enemy)
{
    enemy.TakeDamage(10);
}
// Can only attack enemies. What about barrels? Players?

// With interface — decoupled
public interface IDamageable
{
    void TakeDamage(int amount);
}

void Attack(IDamageable target)
{
    target.TakeDamage(10);
}
// Can attack anything damageable
```

### Implementation

```csharp
public interface IDamageable
{
    int Health { get; }
    void TakeDamage(int amount);
}

public interface IInteractable
{
    string GetPrompt();
    void Interact(GameObject interactor);
}

// Implement multiple interfaces
public class Enemy : MonoBehaviour, IDamageable, IInteractable
{
    public int Health { get; private set; } = 100;
    
    public void TakeDamage(int amount)
    {
        Health -= amount;
        if (Health <= 0) Die();
    }
    
    public string GetPrompt() => "Attack";
    public void Interact(GameObject interactor) => TakeDamage(10);
}

public class Barrel : MonoBehaviour, IDamageable
{
    public int Health { get; private set; } = 50;
    
    public void TakeDamage(int amount)
    {
        Health -= amount;
        if (Health <= 0) Explode();
    }
}
```

### Using Interfaces

```csharp
// Check and use
if (collision.gameObject.TryGetComponent<IDamageable>(out var target))
{
    target.TakeDamage(damage);
}

// Find all damageable in range
var hits = Physics.OverlapSphere(center, radius);
foreach (var hit in hits)
{
    if (hit.TryGetComponent<IDamageable>(out var target))
        target.TakeDamage(damage);
}
```

### Default Interface Methods (C# 8+)

```csharp
public interface ILogger
{
    void Log(string message);
    
    // Default implementation — can be overridden
    void LogWarning(string message) => Log($"WARNING: {message}");
    void LogError(string message) => Log($"ERROR: {message}");
}
```

---

## Enums

Named constants. Readable, type-safe alternatives to magic numbers.

```csharp
public enum EnemyState
{
    Idle,       // 0
    Patrol,     // 1
    Chase,      // 2
    Attack      // 3
}

// Usage
EnemyState state = EnemyState.Idle;

if (state == EnemyState.Chase)
    MoveTowardPlayer();

// Switch
switch (state)
{
    case EnemyState.Idle: /* ... */ break;
    case EnemyState.Chase: /* ... */ break;
}

// Switch expression (cleaner)
string animation = state switch
{
    EnemyState.Idle => "idle",
    EnemyState.Patrol => "walk",
    EnemyState.Chase => "run",
    EnemyState.Attack => "attack",
    _ => "idle"
};
```

### Explicit Values

```csharp
public enum DamageType
{
    Physical = 0,
    Fire = 10,
    Ice = 20,
    Lightning = 30
}

// Useful for:
// - Database storage
// - Network sync
// - Gaps for future additions
```

### Flags Enum

For combining multiple values.

```csharp
[Flags]
public enum Abilities
{
    None = 0,
    CanFly = 1,         // 0001
    CanSwim = 2,        // 0010
    CanBurrow = 4,      // 0100
    CanTeleport = 8,    // 1000
    All = CanFly | CanSwim | CanBurrow | CanTeleport
}

// Combine
Abilities abilities = Abilities.CanFly | Abilities.CanSwim;

// Check
if (abilities.HasFlag(Abilities.CanFly))
    Fly();

// Add
abilities |= Abilities.CanBurrow;

// Remove
abilities &= ~Abilities.CanFly;
```

### Conversion

```csharp
// Enum to int
int value = (int)EnemyState.Chase;    // 2

// Int to enum
EnemyState state = (EnemyState)2;     // Chase

// String to enum
EnemyState state = Enum.Parse<EnemyState>("Chase");
if (Enum.TryParse<EnemyState>("Chase", out var state)) { }

// Enum to string
string name = EnemyState.Chase.ToString();    // "Chase"

// Get all values
foreach (EnemyState state in Enum.GetValues<EnemyState>())
    Debug.Log(state);
```

---

## Tuples

Return or group multiple values without creating a class.

### Why Tuples

```csharp
// Without tuples — need a class or out parameters
class MinMaxResult { public int Min; public int Max; }
MinMaxResult GetMinMax(int[] arr) { ... }

// Or ugly out parameters
void GetMinMax(int[] arr, out int min, out int max) { ... }

// With tuples — clean and simple
(int min, int max) GetMinMax(int[] arr)
{
    return (arr.Min(), arr.Max());
}

var result = GetMinMax(numbers);
Console.WriteLine(result.min);

// Or deconstruct directly
var (min, max) = GetMinMax(numbers);
```

### Syntax

```csharp
// Unnamed tuple
(int, string) data = (1, "hello");
int num = data.Item1;
string str = data.Item2;

// Named tuple (preferred)
(int count, string name) data = (1, "hello");
int num = data.count;

// Return from method
(int x, int y) GetPosition() => (10, 20);

// Deconstruction
var (x, y) = GetPosition();

// Discard values you don't need
var (_, y) = GetPosition();   // Ignore x
```

**When to use:**

- Return multiple values from a method
- Temporary grouping of values
- Not worth creating a class/struct for

**When NOT to use:**

- Public API (use a proper class/struct for clarity)
- Complex data (more than 3-4 values)
- Reused data shape (create a type)

---

## Pattern Matching

Powerful conditionals based on shape/type of data.

### Type Patterns

```csharp
// Old way
if (obj is Enemy)
{
    Enemy enemy = (Enemy)obj;
    enemy.TakeDamage(10);
}

// Pattern matching — combined check and cast
if (obj is Enemy enemy)
{
    enemy.TakeDamage(10);
}

// With additional conditions
if (obj is Enemy enemy && enemy.Health > 0)
{
    enemy.TakeDamage(10);
}

// Negation
if (obj is not null)
{
    // Use obj
}
```

### Switch Expressions

```csharp
// Old switch statement
string GetDescription(EnemyState state)
{
    switch (state)
    {
        case EnemyState.Idle: return "Standing";
        case EnemyState.Chase: return "Pursuing";
        default: return "Unknown";
    }
}

// Switch expression — cleaner
string GetDescription(EnemyState state) => state switch
{
    EnemyState.Idle => "Standing",
    EnemyState.Patrol => "Walking",
    EnemyState.Chase => "Pursuing",
    EnemyState.Attack => "Fighting",
    _ => "Unknown"   // Default case
};
```

### Property Patterns

```csharp
// Match based on properties
if (enemy is { Health: > 0, IsBoss: true })
{
    // Alive boss
}

// In switch
string GetThreatLevel(Enemy enemy) => enemy switch
{
    { Health: <= 0 } => "Dead",
    { IsBoss: true, Health: > 50 } => "Dangerous",
    { IsBoss: true } => "Weakened Boss",
    { Health: > 75 } => "Healthy",
    { Health: > 25 } => "Wounded",
    _ => "Critical"
};
```

### Relational & Logical Patterns

```csharp
// Relational (comparing values)
string GetGrade(int score) => score switch
{
    >= 90 => "A",
    >= 80 => "B",
    >= 70 => "C",
    >= 60 => "D",
    _ => "F"
};

// Logical (and, or, not)
if (health is > 0 and <= 25)
    ShowLowHealthWarning();

if (state is EnemyState.Idle or EnemyState.Patrol)
    // Not aggressive

if (target is not null)
    // Safe to use
```

---

## Null Handling

C#'s tools for dealing with null safely.

### The Problem

```csharp
Enemy enemy = GetEnemy();     // Might return null
enemy.TakeDamage(10);         // NullReferenceException if null
```

### Null Conditional (?.)

```csharp
// Old way
if (enemy != null)
    enemy.TakeDamage(10);

// Null conditional — short-circuit if null
enemy?.TakeDamage(10);

// Chaining
string name = enemy?.Weapon?.Name;   // null if any part is null

// With method calls
int? length = enemy?.Name?.Length;
```

### Null Coalescing (??)

```csharp
// Provide default if null
string name = enemy?.Name ?? "Unknown";
int health = enemy?.Health ?? 0;

// Chain of fallbacks
string displayName = customName ?? enemy?.Name ?? "Enemy";
```

### Null Coalescing Assignment (??=)

```csharp
// Assign only if currently null
_cachedEnemy ??= FindEnemy();

// Equivalent to:
if (_cachedEnemy == null)
    _cachedEnemy = FindEnemy();
```

### Null Forgiving (!)

```csharp
// Tells compiler "trust me, this isn't null"
string name = enemy!.Name;   // Suppresses warning

// Use sparingly — you're bypassing safety
// Only when you KNOW it's not null but compiler doesn't
```

### Patterns for Null

```csharp
// These are equivalent but patterns are clearer
if (enemy is not null) { }
if (enemy != null) { }

// Especially useful in switch
string status = enemy switch
{
    null => "No enemy",
    { Health: <= 0 } => "Dead",
    _ => "Alive"
};
```

---

## Delegates and Events

### Delegates: Function Pointers

Delegates hold references to methods.

```csharp
// Declare delegate type (rarely needed — use built-in)
public delegate void DamageHandler(int amount);

// Built-in delegates (use these)
Action                          // void Method()
Action<int>                     // void Method(int)
Action<int, string>             // void Method(int, string)
Func<int>                       // int Method()
Func<int, string>               // string Method(int)
Func<int, int, bool>            // bool Method(int, int)

// Usage
Action<int> callback = SomeMethod;
callback(42);   // Calls SomeMethod(42)

// Multicast — multiple methods
callback += AnotherMethod;
callback(42);   // Calls both methods
```

### Events: Safe Delegates

Events are delegates with restrictions — only owner can invoke.

```csharp
public class Health : MonoBehaviour
{
    // Declare event
    public event Action OnDied;
    public event Action<int, int> OnChanged;  // current, max
    
    private int _current;
    
    public void TakeDamage(int amount)
    {
        _current -= amount;
        OnChanged?.Invoke(_current, _max);   // ?. in case no subscribers
        
        if (_current <= 0)
            OnDied?.Invoke();
    }
}

// Subscribing (outside the class)
health.OnDied += HandleDeath;
health.OnChanged += HandleHealthChange;

// Unsubscribing (IMPORTANT — prevents memory leaks)
health.OnDied -= HandleDeath;
health.OnChanged -= HandleHealthChange;

// Handler methods
void HandleDeath()
{
    Debug.Log("Died!");
}

void HandleHealthChange(int current, int max)
{
    healthBar.fillAmount = (float)current / max;
}
```

### Why Events Over Public Delegates

```csharp
// Delegate: anyone can invoke or replace
public Action OnDied;
enemy.OnDied = null;        // Oops, cleared all handlers
enemy.OnDied();             // Anyone can fake a death

// Event: only owner can invoke
public event Action OnDied;
enemy.OnDied = null;        // ERROR
enemy.OnDied();             // ERROR
enemy.OnDied += Handler;    // OK — can only add/remove
```

### Subscribe/Unsubscribe Pattern

**Always unsubscribe to prevent memory leaks and errors.**

```csharp
public class HealthUI : MonoBehaviour
{
    [SerializeField] private Health _health;
    
    private void OnEnable()
    {
        _health.OnChanged += UpdateDisplay;
        _health.OnDied += HandleDeath;
    }
    
    private void OnDisable()
    {
        _health.OnChanged -= UpdateDisplay;
        _health.OnDied -= HandleDeath;
    }
    
    private void UpdateDisplay(int current, int max) { /* ... */ }
    private void HandleDeath() { /* ... */ }
}
```

---

## Lambdas

Anonymous inline functions.

### Syntax

```csharp
// Expression lambda (single expression)
Func<int, int> square = x => x * x;
Func<int, int, int> add = (a, b) => a + b;
Action<string> log = msg => Debug.Log(msg);

// Statement lambda (multiple statements)
Action<int> process = x =>
{
    int doubled = x * 2;
    Debug.Log(doubled);
    SaveValue(doubled);
};
```

### Common Uses

```csharp
// LINQ
enemies.Where(e => e.Health > 0);
enemies.OrderBy(e => e.Health);
enemies.Select(e => e.Name);

// Event handlers
button.onClick.AddListener(() => Debug.Log("Clicked!"));
button.onClick.AddListener(() =>
{
    PlaySound();
    LoadScene("Game");
});

// Callbacks
StartCoroutine(FadeOut(() => Destroy(gameObject)));

// Sorting
enemies.Sort((a, b) => a.Health.CompareTo(b.Health));
```

### Captured Variables

Lambdas can use variables from enclosing scope.

```csharp
int multiplier = 10;
Func<int, int> multiply = x => x * multiplier;   // Captures 'multiplier'

multiply(5);    // 50
multiplier = 20;
multiply(5);    // 100 — uses current value
```

**Gotcha — loop variable capture:**

```csharp
// WRONG — all callbacks use final value of i
for (int i = 0; i < 5; i++)
    buttons[i].onClick.AddListener(() => Debug.Log(i));   // All print 5!

// RIGHT — capture a copy
for (int i = 0; i < 5; i++)
{
    int index = i;   // Local copy
    buttons[i].onClick.AddListener(() => Debug.Log(index));
}
```

---

## Generics

Write code that works with any type.

### Why Generics

```csharp
// Without generics — duplicate code or lose type safety
class IntList { public void Add(int item) { } }
class StringList { public void Add(string item) { } }
// Or use object and cast everywhere

// With generics — one implementation, any type
class List<T> { public void Add(T item) { } }

List<int> numbers = new List<int>();
List<string> names = new List<string>();
List<Enemy> enemies = new List<Enemy>();
```

### Generic Methods

```csharp
// Works with any type
public T GetRandom<T>(List<T> list)
{
    return list[Random.Range(0, list.Count)];
}

Enemy enemy = GetRandom(enemies);
string name = GetRandom(names);
```

### Generic Classes

```csharp
public class ObjectPool<T> where T : Component
{
    private Queue<T> _pool = new();
    private T _prefab;
    
    public ObjectPool(T prefab, int initialSize)
    {
        _prefab = prefab;
        for (int i = 0; i < initialSize; i++)
            _pool.Enqueue(CreateNew());
    }
    
    private T CreateNew() => Object.Instantiate(_prefab);
    
    public T Get() => _pool.Count > 0 ? _pool.Dequeue() : CreateNew();
    public void Return(T obj) => _pool.Enqueue(obj);
}

// Usage
var bulletPool = new ObjectPool<Bullet>(bulletPrefab, 20);
var enemyPool = new ObjectPool<Enemy>(enemyPrefab, 10);
```

### Constraints

Limit what types can be used.

```csharp
// Must be a reference type
public class Cache<T> where T : class { }

// Must be a value type
public class Buffer<T> where T : struct { }

// Must have parameterless constructor
public class Factory<T> where T : new()
{
    public T Create() => new T();
}

// Must inherit from / implement
public class EnemyPool<T> where T : Enemy { }
public class Pool<T> where T : Component { }
public class DamageDealer<T> where T : IDamageable { }

// Multiple constraints
public class Pool<T> where T : Component, new() { }
```

---

## Extension Methods

Add methods to existing types without modifying them.

```csharp
public static class FloatExtensions
{
    // 'this' keyword makes it an extension method
    public static float Remap(this float value, float fromMin, float fromMax, float toMin, float toMax)
    {
        return (value - fromMin) / (fromMax - fromMin) * (toMax - toMin) + toMin;
    }
    
    public static bool Approximately(this float a, float b)
    {
        return Mathf.Approximately(a, b);
    }
}

// Usage — looks like instance method
float normalized = health.Remap(0, 100, 0, 1);
if (value.Approximately(target)) { }
```

### Unity Extensions

```csharp
public static class TransformExtensions
{
    public static void Reset(this Transform t)
    {
        t.localPosition = Vector3.zero;
        t.localRotation = Quaternion.identity;
        t.localScale = Vector3.one;
    }
    
    public static void DestroyChildren(this Transform t)
    {
        for (int i = t.childCount - 1; i >= 0; i--)
            Object.Destroy(t.GetChild(i).gameObject);
    }
    
    public static T GetOrAddComponent<T>(this GameObject go) where T : Component
    {
        return go.TryGetComponent<T>(out var component) ? component : go.AddComponent<T>();
    }
}

// Usage
transform.Reset();
container.DestroyChildren();
var rb = gameObject.GetOrAddComponent<Rigidbody>();
```

```csharp
public static class ListExtensions
{
    public static T GetRandom<T>(this List<T> list)
    {
        return list[Random.Range(0, list.Count)];
    }
    
    public static void Shuffle<T>(this List<T> list)
    {
        for (int i = list.Count - 1; i > 0; i--)
        {
            int j = Random.Range(0, i + 1);
            (list[i], list[j]) = (list[j], list[i]);
        }
    }
}

// Usage
Enemy randomEnemy = enemies.GetRandom();
cards.Shuffle();
```

---

## Async / Await

Handle asynchronous operations without blocking.

### The Problem

```csharp
// This blocks the main thread — game freezes
string data = DownloadFile(url);   // Waits here until complete
ProcessData(data);
```

### The Solution

```csharp
// This doesn't block — game continues running
async Task DownloadAndProcessAsync()
{
    string data = await DownloadFileAsync(url);   // Yields control
    ProcessData(data);                             // Continues when ready
}
```

### Basic Syntax

```csharp
// Async method that returns a value
public async Task<string> FetchDataAsync()
{
    using var client = new HttpClient();
    string result = await client.GetStringAsync(url);
    return result;
}

// Async method that returns nothing
public async Task ProcessAsync()
{
    await Task.Delay(1000);   // Wait 1 second
    DoSomething();
}

// Calling async methods
string data = await FetchDataAsync();
await ProcessAsync();
```

### async void — Use Sparingly

```csharp
// async void only for event handlers
public async void OnButtonClicked()
{
    await DoSomethingAsync();
}

// Why avoid async void:
// - Exceptions can't be caught
// - Can't await it
// - Can't know when it completes

// Prefer async Task
public async Task OnButtonClickedAsync()
{
    await DoSomethingAsync();
}
```

### Parallel Execution

```csharp
// Sequential — slow
var data1 = await FetchData1Async();   // Wait
var data2 = await FetchData2Async();   // Then wait

// Parallel — fast
var task1 = FetchData1Async();   // Start
var task2 = FetchData2Async();   // Start
await Task.WhenAll(task1, task2);   // Wait for both
var data1 = task1.Result;
var data2 = task2.Result;
```

### In Unity

Standard `Task` can have issues with Unity's threading model. Use:

1. **Coroutines** for simple delays and sequences
2. **UniTask** (package) for proper async in Unity

```csharp
// UniTask example
using Cysharp.Threading.Tasks;

async UniTaskVoid Start()
{
    await UniTask.Delay(1000);   // 1 second, no GC allocation
    await SceneManager.LoadSceneAsync("Game");
    
    // Cancellation
    var token = this.GetCancellationTokenOnDestroy();
    await UniTask.Delay(1000, cancellationToken: token);
}
```

---

## Parameter Modifiers: ref, out, in

By default, parameters are passed **by value** — a copy is made.

### No Modifier (Default)

```csharp
void Modify(int x)
{
    x = 100;    // Modifies the COPY
}

int num = 5;
Modify(num);
// num is still 5
```

### ref — Two-Way Reference

Pass a reference to the variable. Changes affect the original. **Must be initialized before passing.**

```csharp
void Modify(ref int x)
{
    x = 100;    // Modifies the ORIGINAL
}

int num = 5;
Modify(ref num);    // Must use 'ref' keyword at call site
// num is now 100
```

**When to use:**

- Method needs to modify caller's variable
- Avoid copying large structs (performance)

```csharp
// Performance: avoid copying large struct
void UpdateTransform(ref Matrix4x4 matrix)
{
    // Work directly with original, no copy
}

// Swap values
void Swap<T>(ref T a, ref T b)
{
    T temp = a;
    a = b;
    b = temp;
}
```

### out — Output Only aka producer

Like `ref`, but **must be assigned inside the method**. Doesn't need initialization before passing.

```csharp
void GetValues(out int x, out int y)
{
    x = 10;     // MUST assign before method returns
    y = 20;
}

GetValues(out int a, out int b);    // Inline declaration
// a = 10, b = 20

// Or with existing variables
int a, b;
GetValues(out a, out b);
```

**When to use:**

- Return multiple values (alternative to tuples)
- "Try" pattern — return success + result

```csharp
// The classic TryParse pattern
if (int.TryParse(input, out int result))
{
    // Use result
}

// TryGetValue pattern
if (dict.TryGetValue("key", out var value))
{
    // Use value
}

// Discard with _
if (int.TryParse(input, out _))    // Don't care about the value
{
    // Just checking if valid
}
```

### in — Read-Only Reference aka consumer

Pass by reference but **cannot modify**. Compiler enforces immutability.

```csharp
void PrintLength(in Vector3 v)
{
    // v.x = 10;    // ERROR: cannot modify
    Debug.Log(v.magnitude);    // OK: reading
}

Vector3 pos = new Vector3(1, 2, 3);
PrintLength(in pos);    // 'in' keyword optional at call site
PrintLength(pos);       // Also works
```

**When to use:**

- Large structs you only need to read
- Guarantee method won't modify the value
- Performance: avoid copying large value types

```csharp
// Good for large readonly structs
void ProcessMatrix(in Matrix4x4 matrix) { }
void CheckBounds(in Bounds bounds) { }

// NOT useful for small types (int, float) — overhead isn't worth it
```

### Summary Table

|Modifier|Initialized Before?|Must Assign Inside?|Can Modify?|Use Case|
|---|---|---|---|---|
|(none)|Yes|No|Copy only|Default, safe|
|`ref`|Yes|No|Yes|Modify original|
|`out`|No|Yes|Yes|Return multiple values|
|`in`|Yes|No|No (read-only)|Large struct, readonly|

---

## params — Variable Arguments

Accept any number of arguments as an array.

```csharp
void Log(params string[] messages)
{
    foreach (var msg in messages)
        Debug.Log(msg);
}

// Call with any number of arguments
Log("one");
Log("one", "two", "three");
Log();    // Zero arguments OK

// Or pass an array directly
string[] arr = { "a", "b", "c" };
Log(arr);
```

**Rules:**

- Must be last parameter
- Only one `params` per method

**When to use:**

- Convenience methods accepting variable input
- String formatting, logging

```csharp
// Common example: string.Format
string.Format("{0} + {1} = {2}", a, b, a + b);

// Your own methods
void SpawnEnemies(params EnemyType[] types)
{
    foreach (var type in types)
        Spawn(type);
}

SpawnEnemies(EnemyType.Goblin, EnemyType.Orc, EnemyType.Troll);
```

---

## Indexers

Make your class accessible with `[]` syntax like an array.

```csharp
public class Inventory
{
    private Item[] _items = new Item[10];
    
    // Indexer
    public Item this[int index]
    {
        get => _items[index];
        set => _items[index] = value;
    }
}

// Usage
var inventory = new Inventory();
inventory[0] = sword;           // Set via indexer
Item item = inventory[0];       // Get via indexer
```

**Multiple indexers with different types:**

```csharp
public class Registry
{
    private Dictionary<string, Item> _byName = new();
    private Dictionary<int, Item> _byId = new();
    
    public Item this[string name] => _byName[name];
    public Item this[int id] => _byId[id];
}

var item1 = registry["Sword"];    // By name
var item2 = registry[42];         // By ID
```

---

## Operator Overloading

Define how operators work with your types.

```csharp
public struct Money
{
    public decimal Amount { get; }
    public string Currency { get; }
    
    public Money(decimal amount, string currency)
    {
        Amount = amount;
        Currency = currency;
    }
    
    // Addition
    public static Money operator +(Money a, Money b)
    {
        if (a.Currency != b.Currency)
            throw new InvalidOperationException("Currency mismatch");
        return new Money(a.Amount + b.Amount, a.Currency);
    }
    
    // Subtraction
    public static Money operator -(Money a, Money b)
    {
        if (a.Currency != b.Currency)
            throw new InvalidOperationException("Currency mismatch");
        return new Money(a.Amount - b.Amount, a.Currency);
    }
    
    // Comparison
    public static bool operator ==(Money a, Money b)
        => a.Currency == b.Currency && a.Amount == b.Amount;
    
    public static bool operator !=(Money a, Money b)
        => !(a == b);
    
    // Scalar multiplication
    public static Money operator *(Money m, decimal multiplier)
        => new Money(m.Amount * multiplier, m.Currency);
}

// Usage
var price = new Money(10, "USD");
var tax = new Money(2, "USD");
var total = price + tax;             // Money(12, "USD")
var doubled = price * 2;             // Money(20, "USD")
```

**Common operators to overload:**

- Arithmetic: `+`, `-`, `*`, `/`, `%`
- Comparison: `==`, `!=`, `<`, `>`, `<=`, `>=`
- Unary: `+`, `-`, `!`, `++`, `--`
- Implicit/explicit conversion

**Unity example — why Vector3 works with +:**

```csharp
// Unity defines this internally
public static Vector3 operator +(Vector3 a, Vector3 b)
    => new Vector3(a.x + b.x, a.y + b.y, a.z + b.z);

// So you can do
Vector3 position = startPos + offset;
```

---

## Implicit and Explicit Conversions

Define how your type converts to/from other types.

```csharp
public struct Percentage
{
    public float Value { get; }
    
    public Percentage(float value) => Value = value;
    
    // Implicit: automatic, no cast needed (safe conversion)
    public static implicit operator float(Percentage p) => p.Value;
    
    // Explicit: requires cast (might lose data)
    public static explicit operator Percentage(float f) => new Percentage(f);
}

// Usage
Percentage health = (Percentage)0.75f;   // Explicit: need cast
float value = health;                     // Implicit: no cast needed
```

**When to use implicit:** Conversion is always safe, no data loss. **When to use explicit:** Conversion might fail or lose precision.

---

## Disposable Pattern (IDisposable, using)

Clean up unmanaged resources (files, network, etc.).

### Why It Exists

```csharp
// Problem: resource leak
var file = File.OpenRead("data.txt");
ProcessFile(file);
// Forgot to close! File stays locked.
```

### using Statement

```csharp
// Automatic cleanup when scope exits
using (var file = File.OpenRead("data.txt"))
{
    ProcessFile(file);
}   // file.Dispose() called automatically, even if exception

// Modern syntax (C# 8+) — disposed at end of enclosing scope
using var file = File.OpenRead("data.txt");
ProcessFile(file);
// Disposed when method returns
```

### Implementing IDisposable

```csharp
public class ResourceHolder : IDisposable
{
    private FileStream _file;
    private bool _disposed = false;
    
    public ResourceHolder(string path)
    {
        _file = File.OpenRead(path);
    }
    
    public void Dispose()
    {
        if (_disposed) return;
        
        _file?.Dispose();
        _disposed = true;
    }
}

// Usage
using var holder = new ResourceHolder("data.txt");
```

**Common disposables in Unity:**

- `WWW` / `UnityWebRequest` (use `using`)
- `NativeArray<T>` (must dispose!)
- Custom native plugins

---

## Static Classes and Members

### Static Members

Belong to the type, not instances.

```csharp
public class Enemy
{
    // Instance member — each enemy has its own
    public int Health { get; set; }
    
    // Static member — shared across all enemies
    public static int TotalCount { get; private set; }
    
    public Enemy()
    {
        TotalCount++;
    }
}

// Access static via type name
Debug.Log(Enemy.TotalCount);

// Instance via object
var goblin = new Enemy();
Debug.Log(goblin.Health);
```

### Static Classes

Cannot be instantiated. All members must be static.

```csharp
public static class MathUtils
{
    public static float Remap(float value, float fromMin, float fromMax, float toMin, float toMax)
    {
        return (value - fromMin) / (fromMax - fromMin) * (toMax - toMin) + toMin;
    }
    
    public static int RandomSign() => Random.value > 0.5f ? 1 : -1;
}

// Usage
float normalized = MathUtils.Remap(health, 0, 100, 0, 1);
```

**When to use static classes:**

- Utility/helper methods
- Extension method containers
- Pure functions with no state

**When NOT to use:**

- When you need instances (testability, dependency injection)
- Game state (harder to reset, test, mock)

---

## Namespaces

Organize code and prevent naming conflicts.

```csharp
namespace MyGame.Combat
{
    public class DamageSystem { }
}

namespace MyGame.UI
{
    public class DamagePopup { }
}

// Usage
var system = new MyGame.Combat.DamageSystem();
var popup = new MyGame.UI.DamagePopup();

// Or with using
using MyGame.Combat;
using MyGame.UI;

var system = new DamageSystem();
var popup = new DamagePopup();
```

### File-Scoped Namespaces (C# 10)

```csharp
// Old way
namespace MyGame.Combat
{
    public class DamageSystem
    {
        // Everything indented
    }
}

// New way — no extra indentation
namespace MyGame.Combat;

public class DamageSystem
{
    // Clean
}
```

### Using Aliases

```csharp
using Vec3 = UnityEngine.Vector3;
using Dict = System.Collections.Generic.Dictionary<string, int>;

Vec3 position = Vec3.zero;
Dict scores = new Dict();
```

### Global Using (C# 10)

```csharp
// In a central file, applies to whole project
global using UnityEngine;
global using System.Collections.Generic;
```

---

## Attributes

Metadata attached to code elements. Affect compilation or runtime behavior.

### Common Unity Attributes

```csharp
// Serialization
[SerializeField] private int _health;        // Show private in Inspector
[HideInInspector] public int dontShow;       // Hide public in Inspector
[field: SerializeField] public int Prop { get; private set; }  // Property

// Inspector organization
[Header("Movement")]
[SerializeField] private float _speed;

[Tooltip("Units per second")]
[SerializeField] private float _velocity;

[Space(10)]
[SerializeField] private float _other;

[Range(0, 100)]
[SerializeField] private int _percentage;

[TextArea(3, 5)]
[SerializeField] private string _description;

// Component requirements
[RequireComponent(typeof(Rigidbody))]
public class Movement : MonoBehaviour { }

[DisallowMultipleComponent]
public class GameManager : MonoBehaviour { }

// Execution
[ExecuteInEditMode]                          // Runs in editor
[ExecuteAlways]                              // Runs in editor and play
[DefaultExecutionOrder(-100)]                // Script ordering

// ScriptableObject
[CreateAssetMenu(fileName = "New Item", menuName = "Game/Item")]
public class ItemData : ScriptableObject { }
```

### Common C# Attributes

```csharp
[Obsolete("Use NewMethod instead")]
public void OldMethod() { }

[Obsolete("Removed in v2.0", error: true)]   // Compile error
public void ReallyOldMethod() { }

[Conditional("DEBUG")]                        // Only compiled in debug
public void DebugLog(string msg) { }

[System.Serializable]                         // For Unity/JSON serialization
public class SaveData { }

[System.NonSerialized]                        // Exclude from serialization
public int tempValue;
```

### Custom Attributes

```csharp
// Define
[AttributeUsage(AttributeTargets.Field)]
public class MinValueAttribute : PropertyAttribute
{
    public float Min { get; }
    public MinValueAttribute(float min) => Min = min;
}

// Use
[MinValue(0)]
public float health;

// Would need custom PropertyDrawer to enforce in Inspector
```

---

## Preprocessor Directives

Conditional compilation based on symbols.

```csharp
// Platform-specific code
#if UNITY_EDITOR
    Debug.Log("Editor only");
#elif UNITY_ANDROID
    Debug.Log("Android build");
#elif UNITY_IOS
    Debug.Log("iOS build");
#else
    Debug.Log("Other platform");
#endif

// Debug-only code
#if DEBUG
    ValidateState();
#endif

// Define your own symbols in Player Settings
#if MY_FEATURE_ENABLED
    EnableFeature();
#endif

// Disable code without deleting
#if false
    // This code won't compile
    OldImplementation();
#endif

// Warning/Error
#warning This is incomplete
#error Do not ship with this code
```

**Common Unity symbols:**

- `UNITY_EDITOR` — running in editor
- `UNITY_ANDROID`, `UNITY_IOS`, `UNITY_STANDALONE`
- `UNITY_2021_1_OR_NEWER` — version checks
- `DEBUG` — debug build
- `DEVELOPMENT_BUILD` — development build

---

## Exception Handling

### try / catch / finally

```csharp
try
{
    RiskyOperation();
}
catch (FileNotFoundException ex)
{
    // Handle specific exception
    Debug.LogError($"File not found: {ex.FileName}");
}
catch (Exception ex) when (ex.Message.Contains("network"))
{
    // Filtered catch — only catches if condition true
    Debug.LogError("Network error");
}
catch (Exception ex)
{
    // Catch all others
    Debug.LogError($"Unexpected error: {ex.Message}");
    throw;    // Re-throw preserving stack trace
}
finally
{
    // ALWAYS runs, even if exception thrown
    CleanUp();
}
```

### Throwing Exceptions

```csharp
// Throw built-in exceptions
throw new ArgumentNullException(nameof(parameter));
throw new ArgumentOutOfRangeException(nameof(index), "Must be positive");
throw new InvalidOperationException("Cannot perform action in current state");
throw new NotImplementedException();
throw new NotSupportedException();

// Custom exception
public class GameException : Exception
{
    public int ErrorCode { get; }
    
    public GameException(string message, int code) : base(message)
    {
        ErrorCode = code;
    }
}

throw new GameException("Save failed", 101);
```

### When NOT to Use Exceptions

Exceptions are expensive. Don't use for normal control flow.

```csharp
// BAD: Using exception for normal case
try
{
    var item = dict[key];
}
catch (KeyNotFoundException)
{
    // Handle missing key
}

// GOOD: Check first
if (dict.TryGetValue(key, out var item))
{
    // Use item
}
else
{
    // Handle missing
}
```

---

## Covariance and Contravariance

How inheritance works with generics.

TLDR; `out` means for output and therefore it needs strip down safety. `in` means input therefore implementation needs strip down safety

### Covariance (out) — Can Return Derived Types

> Producer widens the gates and catch as wide as we need, as usage will strip down as it needs.

```csharp
// IEnumerable<out T> is the classic producer interface
IEnumerable<string> strings = new List<string> { "Hello" };

// Safe: You can treat a list of strings as a list of objects 
// because every item you pull 'out' is guaranteed to be an object.
IEnumerable<object> objects = strings; 

foreach (object obj in objects) {
    Console.WriteLine(obj.ToString());
}
```

If implementation states it produces (outputting data `out`) `Dog`s, it's safe to treat it as producing `Animal`s, because every `Dog` is an `Animal`, and returning type of the results of `IEnumerable` will return any of the `Animal`

implementation set to return `Dog` -> the usage catches _any_ `Animal` | END

### Contravariance (in) — Can Accept Base Types (consumer)

> Consumer. tightens the gates as the implementation will strip down to what it can

```csharp
// Action<in T> is a classic consumer delegate
Action<object> printObject = (obj) => Console.WriteLine(obj.GetHashCode());

// Safe: You can treat an 'object' printer as a 'string' printer.
// The code expecting a string-printer will only ever pass in strings;
// since a string IS an object, the printObject logic handles it perfectly.
Action<string> printString = printObject;

printString("World"); 
```

If implementation states it consumes (and doing something with it readonly `in`) `Animal`, it is allowed since interface states that using it can handle _up to_ `Dog`s, therefore the implementation itself only will use that part that is == or < than the ceiling.

implementation set to handle `Animal` -> the usage passing `Dog` | END

### Practical Example

```csharp
// You have a method that processes animals
void ProcessAnimals(IEnumerable<Animal> animals) { }

// You can pass a list of dogs
List<Dog> dogs = new List<Dog>();
ProcessAnimals(dogs);  // Works because IEnumerable<T> is covariant
```

---

## Common Gotchas

### Float Comparison

```csharp
// WRONG — floats are imprecise
if (a == b) { }
if (a == 0.3f) { }   // 0.1f + 0.2f != 0.3f

// RIGHT
if (Mathf.Approximately(a, b)) { }
if (Mathf.Abs(a - b) < 0.0001f) { }
```

### Reference Equality vs Value Equality

```csharp
// Reference types: == compares references by default
Enemy a = new Enemy("Goblin");
Enemy b = new Enemy("Goblin");
a == b    // false — different objects

// Unless you override Equals or use records
public record EnemyData(string Name, int Health);
var a = new EnemyData("Goblin", 100);
var b = new EnemyData("Goblin", 100);
a == b    // true — records have value equality
```

### String Comparison

```csharp
// Strings are special — compared by value
string a = "hello";
string b = "hello";
a == b    // true

// But watch out for culture issues
"HELLO".ToLower() == "hello"   // Usually true, but not in Turkish!

// Safe comparison
string.Equals(a, b, StringComparison.OrdinalIgnoreCase)
```

### default Keyword

```csharp
int x = default;        // 0
bool b = default;       // false
string s = default;     // null
Enemy e = default;      // null
Vector3 v = default;    // (0,0,0)

// Useful in generics
public T GetValueOrDefault<T>(string key)
{
    return dict.TryGetValue(key, out T val) ? val : default;
}
```

### Captured Variables in Lambdas

```csharp
// WRONG — captures variable, not value
List<Action> actions = new();
for (int i = 0; i < 5; i++)
    actions.Add(() => Debug.Log(i));

foreach (var action in actions)
    action();   // Prints 5, 5, 5, 5, 5

// RIGHT — capture a copy
for (int i = 0; i < 5; i++)
{
    int copy = i;
    actions.Add(() => Debug.Log(copy));
}
```

---

## Quick Reference Table

|Need|Use|
|---|---|
|Store fixed-size data|`T[]` array|
|Store dynamic list|`List<T>`|
|Key-value lookup|`Dictionary<K,V>`|
|Unique values, fast contains|`HashSet<T>`|
|FIFO queue|`Queue<T>`|
|LIFO stack|`Stack<T>`|
|Filter/transform collections|LINQ|
|Return multiple values|Tuple or `out` parameters|
|Modify caller's variable|`ref` parameter|
|Output-only parameter|`out` parameter|
|Read-only reference (perf)|`in` parameter|
|Variable number of args|`params`|
|Define contract|Interface|
|Named constants|Enum|
|Combine flags|`[Flags]` enum|
|Immutable data class|Record|
|Small value type|Struct|
|Function as parameter|`Action<>` or `Func<>`|
|Observable notifications|Event|
|Safe null access|`?.` and `??`|
|Type checking + cast|`is` pattern|
|Add methods to existing types|Extension methods|
|Type-agnostic code|Generics|
|Non-blocking I/O|async/await|
|Array-like access|Indexer `this[int i]`|
|Custom operators|Operator overloading|
|Auto-cleanup resources|`using` + `IDisposable`|
|Conditional compilation|`#if` / `#endif`|
|Metadata on code|Attributes `[...]`|
|Organize code|Namespaces|
|Visible to all|`public`|
|Hidden, this class only|`private`|
|Subclasses only|`protected`|
|Same assembly only|`internal`|
|Allow override|`virtual`|
|Replace base behavior|`override`|
|Force implementation|`abstract`|
|Prevent inheritance|`sealed`|
|Belongs to type|`static`|
|Compile-time constant|`const`|
|Set once (instance)|`readonly`|
|Set once (shared)|`static readonly`|
