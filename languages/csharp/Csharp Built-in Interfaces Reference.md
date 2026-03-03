# C# Built-in Interfaces Reference

_Essential interfaces for Unity development_

---

## Quick Reference

|Interface|Purpose|Unity Relevance|
|---|---|---|
|`IEnumerable<T>`|Iterate over collections|Essential — LINQ, foreach|
|`IEnumerator`|Control iteration state|Essential — Coroutines|
|`IDisposable`|Deterministic cleanup|High — Native resources|
|`IEquatable<T>`|Value-based equality|High — Dictionary keys|
|`IComparable<T>`|Natural ordering|Medium — Sorting|
|`IComparer<T>`|External comparison|Medium — Multiple sort orders|
|`IEqualityComparer<T>`|External equality|Medium — Custom lookups|
|`ICollection<T>`|Countable, modifiable|Low — Abstraction|
|`IList<T>`|Indexed access|Low — Abstraction|
|`IDictionary<K,V>`|Key-value storage|Low — Abstraction|

---

## `IEnumerable<T>` and IEnumerable

### Definition

```csharp
public interface IEnumerable<out T> : IEnumerable
{
    IEnumerator<T> GetEnumerator();
}

public interface IEnumerable
{
    IEnumerator GetEnumerator();
}
```

### Purpose

Represents a sequence that can be iterated. Foundation of all collection iteration and LINQ.

The `out` keyword makes it **covariant**: `IEnumerable<Dog>` can be assigned to `IEnumerable<Animal>`.

### Implementation

```csharp
public class EnemyWave : IEnumerable<Enemy>
{
    private List<Enemy> _enemies = new();

    public IEnumerator<Enemy> GetEnumerator() => _enemies.GetEnumerator();
    
    IEnumerator IEnumerable.GetEnumerator() => GetEnumerator();
}
```

### yield return

Creates a state machine that produces values lazily—computed only when requested.

```csharp
public IEnumerable<Vector3> GetPatrolPoints()
{
    yield return new Vector3(0, 0, 0);
    yield return new Vector3(10, 0, 0);
    yield return new Vector3(10, 0, 10);
    // Values generated one at a time as iteration proceeds
}

public IEnumerable<int> GenerateInfinite()
{
    int i = 0;
    while (true)
        yield return i++;
    // Works because values are lazy—never fully materialized
}
```

### Common Operations

```csharp
// LINQ works on any IEnumerable<T>
var active = enemies.Where(e => e.IsActive);
var sorted = enemies.OrderBy(e => e.Health);
var first = enemies.FirstOrDefault(e => e.IsBoss);
var total = enemies.Sum(e => e.Damage);

// foreach works on any IEnumerable
foreach (var enemy in enemies)
    enemy.TakeDamage(10);
```

> [!warning] Deferred Execution LINQ queries are lazy. Each enumeration re-executes the query. Call `.ToList()` or `.ToArray()` to cache results when iterating multiple times.

---

## `IEnumerator<T>` and IEnumerator

### Definition

```csharp
public interface IEnumerator<out T> : IEnumerator, IDisposable
{
    T Current { get; }
}

public interface IEnumerator
{
    object Current { get; }
    bool MoveNext();
    void Reset();
}
```

### Purpose

Maintains iteration state. `IEnumerable` is "can be iterated"; `IEnumerator` is "the iterator itself"—the cursor.

### Unity Coroutines

Unity coroutines use the **non-generic** `IEnumerator`. Each `yield` suspends execution; Unity resumes based on the yielded value.

```csharp
public IEnumerator SpawnWave(int count)
{
    for (int i = 0; i < count; i++)
    {
        Instantiate(enemyPrefab, GetSpawnPoint(), Quaternion.identity);
        yield return new WaitForSeconds(0.5f);
    }
}

void Start()
{
    StartCoroutine(SpawnWave(10));
}
```

### Yield Instructions

|Yield Value|Behavior|
|---|---|
|`yield return null`|Wait one frame|
|`new WaitForSeconds(n)`|Wait n seconds (scaled time)|
|`new WaitForSecondsRealtime(n)`|Wait n seconds (unscaled)|
|`new WaitForEndOfFrame()`|Wait until rendering complete|
|`new WaitForFixedUpdate()`|Wait for next physics step|
|`new WaitUntil(() => cond)`|Wait until condition true|
|`new WaitWhile(() => cond)`|Wait while condition true|
|`StartCoroutine(other)`|Wait for nested coroutine|

### Coroutine Patterns

```csharp
// Chaining coroutines
public IEnumerator GameSequence()
{
    yield return StartCoroutine(FadeIn());
    yield return StartCoroutine(PlayIntro());
    yield return StartCoroutine(SpawnEnemies());
}

// Coroutine with callback
public IEnumerator LoadAsync(System.Action onComplete)
{
    yield return SceneManager.LoadSceneAsync("Level1");
    onComplete?.Invoke();
}

// Stoppable coroutine
private Coroutine _activeRoutine;

public void StartMoving()
{
    if (_activeRoutine != null)
        StopCoroutine(_activeRoutine);
    _activeRoutine = StartCoroutine(MoveRoutine());
}
```

### Manual Iteration

```csharp
IEnumerator<int> enumerator = GetNumbers().GetEnumerator();
try
{
    while (enumerator.MoveNext())
    {
        int current = enumerator.Current;
        Debug.Log(current);
    }
}
finally
{
    enumerator.Dispose();
}
// foreach does this automatically
```

---

## IDisposable

### Definition

```csharp
public interface IDisposable
{
    void Dispose();
}
```

### Purpose

Deterministic cleanup of unmanaged resources. GC handles managed memory; `IDisposable` handles everything else: file handles, network connections, GPU buffers, native memory.

### The using Statement

Guarantees `Dispose()` is called even if exceptions occur.

```csharp
// Classic syntax
using (var reader = new StreamReader(path))
{
    string content = reader.ReadToEnd();
} // Dispose() called here

// C# 8+ declaration syntax
using var reader = new StreamReader(path);
string content = reader.ReadToEnd();
// Dispose() called at end of enclosing scope
```

### Standard Dispose Pattern

```csharp
public class NativeWrapper : IDisposable
{
    private IntPtr _handle;
    private bool _disposed;

    public NativeWrapper()
    {
        _handle = NativeLib.Allocate();
    }

    public void Dispose()
    {
        Dispose(true);
        GC.SuppressFinalize(this);
    }

    protected virtual void Dispose(bool disposing)
    {
        if (_disposed) return;

        if (disposing)
        {
            // Free managed resources
        }

        // Free unmanaged resources
        if (_handle != IntPtr.Zero)
        {
            NativeLib.Free(_handle);
            _handle = IntPtr.Zero;
        }

        _disposed = true;
    }

    ~NativeWrapper() => Dispose(false);
}
```

### Unity Considerations

> [!info] MonoBehaviour Cleanup `MonoBehaviour` doesn't implement `IDisposable`. Use `OnDestroy()` instead. For non-MonoBehaviour classes, implement `IDisposable` normally.

```csharp
public class GPUParticles : MonoBehaviour
{
    private ComputeBuffer _buffer;

    void Start()
    {
        _buffer = new ComputeBuffer(1000, sizeof(float) * 4);
    }

    void OnDestroy()
    {
        _buffer?.Release(); // Unity's disposal pattern for GPU resources
    }
}

// Non-MonoBehaviour example
public class MeshDataProcessor : IDisposable
{
    private NativeArray<Vector3> _vertices;

    public MeshDataProcessor(int count)
    {
        _vertices = new NativeArray<Vector3>(count, Allocator.Persistent);
    }

    public void Dispose()
    {
        if (_vertices.IsCreated)
            _vertices.Dispose();
    }
}
```

### Unity Types Requiring Disposal

|Type|Disposal Method|
|---|---|
|`ComputeBuffer`|`.Release()`|
|`NativeArray<T>`|`.Dispose()`|
|`NativeList<T>`|`.Dispose()`|
|`RenderTexture`|`.Release()`|
|`Mesh` (runtime created)|`Destroy()`|
|`Material` (runtime created)|`Destroy()`|
|`AsyncOperationHandle`|`.Release()` (Addressables)|

---

## `IEquatable<T>`

### Definition

```csharp
public interface IEquatable<T>
{
    bool Equals(T other);
}
```

### Purpose

Type-safe, allocation-free equality comparison. Essential for dictionary keys and `HashSet<T>` with value types—avoids boxing that `object.Equals()` causes.

### Complete Implementation

Always implement alongside `Object.Equals()`, `Object.GetHashCode()`, and operators.

```csharp
public struct GridPosition : IEquatable<GridPosition>
{
    public int X;
    public int Y;

    public bool Equals(GridPosition other)
    {
        return X == other.X && Y == other.Y;
    }

    public override bool Equals(object obj)
    {
        return obj is GridPosition other && Equals(other);
    }

    public override int GetHashCode()
    {
        return HashCode.Combine(X, Y);
    }

    public static bool operator ==(GridPosition a, GridPosition b) => a.Equals(b);
    public static bool operator !=(GridPosition a, GridPosition b) => !a.Equals(b);
}
```

### GetHashCode Contract

1. **Equal objects must have equal hash codes**
2. Hash codes should be stable for an object's lifetime
3. Use all fields involved in equality comparison
4. Unequal objects _may_ share hash codes (collision is allowed)

```csharp
// Good: uses all equality fields
public override int GetHashCode() => HashCode.Combine(X, Y, Z);

// Pre-.NET Core alternative
public override int GetHashCode()
{
    unchecked
    {
        int hash = 17;
        hash = hash * 31 + X.GetHashCode();
        hash = hash * 31 + Y.GetHashCode();
        return hash;
    }
}
```

### Usage

```csharp
// Efficient dictionary with struct key
Dictionary<GridPosition, TileData> tiles = new();
tiles[new GridPosition { X = 5, Y = 3 }] = data;

// Fast contains check
HashSet<GridPosition> visited = new();
visited.Add(pos);
bool seen = visited.Contains(pos); // O(1)
```

> [!warning] Float Keys `Vector2` and `Vector3` make poor dictionary keys due to floating-point comparison issues. Use `Vector2Int`/`Vector3Int` or implement a custom comparer with tolerance.

---

## `IComparable<T>`

### Definition

```csharp
public interface IComparable<T>
{
    int CompareTo(T other);
}
```

### Return Value Semantics

|Return|Meaning|
|---|---|
|`< 0`|`this` precedes `other`|
|`0`|Same sort position|
|`> 0`|`this` follows `other`|

### Purpose

Defines the **natural** ordering of a type. Enables `Array.Sort()`, `List<T>.Sort()`, and sorted collections.

### Implementation

```csharp
public struct ScoreEntry : IComparable<ScoreEntry>
{
    public string Player;
    public int Score;
    public DateTime Date;

    public int CompareTo(ScoreEntry other)
    {
        // Primary: score descending (higher first)
        int result = other.Score.CompareTo(Score);
        if (result != 0) return result;

        // Secondary: date ascending (earlier first)
        return Date.CompareTo(other.Date);
    }
}
```

### Usage

```csharp
List<ScoreEntry> scores = GetScores();
scores.Sort(); // Uses CompareTo automatically

// Array.Sort also works
ScoreEntry[] array = scores.ToArray();
Array.Sort(array);

// Binary search requires sorted + IComparable
int index = scores.BinarySearch(targetEntry);
```

> [!tip] Consistency with Equals If `a.CompareTo(b) == 0`, then `a.Equals(b)` should ideally return `true`. Inconsistency causes unexpected behavior in sorted collections.

---

## `IComparer<T>`

### Definition

```csharp
public interface IComparer<T>
{
    int Compare(T x, T y);
}
```

### Purpose

External comparison logic. Use when:

- You need multiple sort orders for the same type
- You can't modify the type's source
- Comparison depends on external state

### IComparable vs IComparer

|IComparable<T>|IComparer<T>|
|---|---|
|Implemented by the type|Separate class|
|Natural/default ordering|Alternative orderings|
|`this.CompareTo(other)`|`comparer.Compare(x, y)`|
|One per type|Unlimited per type|

### Implementation

```csharp
public class Enemy
{
    public float Health;
    public float Distance;
    public int Priority;
}

public class ByDistance : IComparer<Enemy>
{
    public int Compare(Enemy x, Enemy y) 
        => x.Distance.CompareTo(y.Distance);
}

public class ByPriorityDescending : IComparer<Enemy>
{
    public int Compare(Enemy x, Enemy y) 
        => y.Priority.CompareTo(x.Priority);
}
```

### Usage

```csharp
List<Enemy> enemies = GetEnemies();

enemies.Sort(new ByDistance());
enemies.Sort(new ByPriorityDescending());

// Inline with Comparer<T>.Create
enemies.Sort(Comparer<Enemy>.Create(
    (a, b) => a.Health.CompareTo(b.Health)));

// SortedSet with custom order
var nearest = new SortedSet<Enemy>(new ByDistance());
```

---

## `IEqualityComparer<T>`

### Definition

```csharp
public interface IEqualityComparer<T>
{
    bool Equals(T x, T y);
    int GetHashCode(T obj);
}
```

### Purpose

External equality logic for dictionaries and hash sets. Use when:

- You need custom equality rules
- Comparing by a subset of properties
- Case-insensitive string keys

### Implementation

```csharp
// Compare enemies by ID only
public class EnemyIdComparer : IEqualityComparer<Enemy>
{
    public bool Equals(Enemy x, Enemy y)
    {
        if (ReferenceEquals(x, y)) return true;
        if (x is null || y is null) return false;
        return x.Id == y.Id;
    }

    public int GetHashCode(Enemy obj) 
        => obj?.Id.GetHashCode() ?? 0;
}

// Case-insensitive strings
public class IgnoreCaseComparer : IEqualityComparer<string>
{
    public bool Equals(string x, string y)
        => string.Equals(x, y, StringComparison.OrdinalIgnoreCase);

    public int GetHashCode(string obj)
        => obj?.ToUpperInvariant().GetHashCode() ?? 0;
}
```

### Usage

```csharp
// Dictionary with custom equality
var enemyData = new Dictionary<Enemy, Data>(new EnemyIdComparer());

// Case-insensitive inventory
var inventory = new Dictionary<string, int>(new IgnoreCaseComparer());
inventory["Sword"] = 1;
bool has = inventory.ContainsKey("SWORD"); // true

// HashSet deduplication by ID
var unique = new HashSet<Enemy>(enemies, new EnemyIdComparer());

// LINQ Distinct with custom equality
var distinctById = enemies.Distinct(new EnemyIdComparer());
```

---

## `ICollection<T>`

### Definition

```csharp
public interface ICollection<T> : IEnumerable<T>
{
    int Count { get; }
    bool IsReadOnly { get; }
    void Add(T item);
    void Clear();
    bool Contains(T item);
    void CopyTo(T[] array, int arrayIndex);
    bool Remove(T item);
}
```

### Purpose

Extends `IEnumerable<T>` with mutation and counting. Base interface for `IList<T>` and `ISet<T>`.

### When to Use

Accept `ICollection<T>` when you need to know the count without enumerating, or need to add/remove items but don't need indexed access.

```csharp
public void ProcessBatch(ICollection<Enemy> enemies)
{
    if (enemies.Count == 0) return;
    
    // Can enumerate and know size upfront
    var buffer = new Enemy[enemies.Count];
    enemies.CopyTo(buffer, 0);
}
```

---

## `IList<T>`

### Definition

```csharp
public interface IList<T> : ICollection<T>
{
    T this[int index] { get; set; }
    int IndexOf(T item);
    void Insert(int index, T item);
    void RemoveAt(int index);
}
```

### Purpose

Indexed access to elements. Implemented by `List<T>`, arrays, and many other collections.

### When to Use

Accept `IList<T>` when you need:

- Random access by index
- Insert/remove at specific positions
- Index-based operations

```csharp
public void ShuffleInPlace<T>(IList<T> list)
{
    for (int i = list.Count - 1; i > 0; i--)
    {
        int j = Random.Range(0, i + 1);
        (list[i], list[j]) = (list[j], list[i]);
    }
}

// Works with List<T>, arrays, etc.
ShuffleInPlace(myList);
ShuffleInPlace(myArray);
```

### IList vs List

|Accept `IList<T>`|Accept `List<T>`|
|---|---|
|More flexible|Access to `List<T>` specific methods|
|Works with arrays|`AddRange`, `FindAll`, `Sort`|
|Better for libraries|Better when you need those methods|

---

## `IDictionary<TKey, TValue>`

### Definition

```csharp
public interface IDictionary<TKey, TValue> : ICollection<KeyValuePair<TKey, TValue>>
{
    TValue this[TKey key] { get; set; }
    ICollection<TKey> Keys { get; }
    ICollection<TValue> Values { get; }
    void Add(TKey key, TValue value);
    bool ContainsKey(TKey key);
    bool Remove(TKey key);
    bool TryGetValue(TKey key, out TValue value);
}
```

### Purpose

Key-value storage abstraction. Accept when you don't care about the concrete dictionary type.

### Common Patterns

```csharp
// Prefer TryGetValue over ContainsKey + indexer
if (dict.TryGetValue(key, out var value))
{
    // Use value
}

// Iterate key-value pairs
foreach (var kvp in dict)
{
    Debug.Log($"{kvp.Key}: {kvp.Value}");
}

// C# 7+ deconstruction
foreach (var (key, value) in dict)
{
    Debug.Log($"{key}: {value}");
}
```

---

## `IReadOnly` Interfaces

### Definitions

```csharp
public interface IReadOnlyCollection<T> : IEnumerable<T>
{
    int Count { get; }
}

public interface IReadOnlyList<T> : IReadOnlyCollection<T>
{
    T this[int index] { get; }
}

public interface IReadOnlyDictionary<TKey, TValue> : IReadOnlyCollection<KeyValuePair<TKey, TValue>>
{
    TValue this[TKey key] { get; }
    IEnumerable<TKey> Keys { get; }
    IEnumerable<TValue> Values { get; }
    bool ContainsKey(TKey key);
    bool TryGetValue(TKey key, out TValue value);
}
```

### Purpose

Express intent that a method won't modify the collection. Provides compile-time safety.

### Usage Pattern

```csharp
public class Inventory
{
    private List<Item> _items = new();

    // Expose as read-only to prevent external modification
    public IReadOnlyList<Item> Items => _items;

    // Internal methods can still modify
    public void AddItem(Item item) => _items.Add(item);
}

// Consumer can read but not modify
IReadOnlyList<Item> items = inventory.Items;
var first = items[0];     // OK
// items.Add(new Item()); // Won't compile
```

> [!warning] Runtime Casting `IReadOnlyList<T>` is an interface, not a wrapper. Callers can cast back to `List<T>` and modify. For true immutability, return a copy or use `ImmutableList<T>`.

---

## `IObservable<T> and IObserver<T>`

### Definitions

```csharp
public interface IObservable<out T>
{
    IDisposable Subscribe(IObserver<T> observer);
}

public interface IObserver<in T>
{
    void OnNext(T value);
    void OnError(Exception error);
    void OnCompleted();
}
```

### Purpose

Push-based notification pattern. Foundation of Reactive Extensions (Rx). `IObservable<T>` is like `IEnumerable<T>` but push instead of pull.

### Basic Implementation

```csharp
public class DamageEventStream : IObservable<DamageEvent>
{
    private List<IObserver<DamageEvent>> _observers = new();

    public IDisposable Subscribe(IObserver<DamageEvent> observer)
    {
        _observers.Add(observer);
        return new Unsubscriber(_observers, observer);
    }

    public void ReportDamage(DamageEvent evt)
    {
        foreach (var observer in _observers)
            observer.OnNext(evt);
    }

    private class Unsubscriber : IDisposable
    {
        private List<IObserver<DamageEvent>> _observers;
        private IObserver<DamageEvent> _observer;

        public Unsubscriber(List<IObserver<DamageEvent>> observers, 
                           IObserver<DamageEvent> observer)
        {
            _observers = observers;
            _observer = observer;
        }

        public void Dispose() => _observers.Remove(_observer);
    }
}
```

> [!tip] UniRx For Unity, use **UniRx** package which provides full Rx implementation with Unity-specific schedulers and operators.

---

## Interface Composition Patterns

### Combining Interfaces

```csharp
public struct EntityId : IEquatable<EntityId>, IComparable<EntityId>
{
    public readonly int Value;

    public EntityId(int value) => Value = value;

    public bool Equals(EntityId other) => Value == other.Value;
    public int CompareTo(EntityId other) => Value.CompareTo(other.Value);

    public override bool Equals(object obj) 
        => obj is EntityId other && Equals(other);
    public override int GetHashCode() => Value;
    
    public static bool operator ==(EntityId a, EntityId b) => a.Equals(b);
    public static bool operator !=(EntityId a, EntityId b) => !a.Equals(b);
    public static bool operator <(EntityId a, EntityId b) => a.CompareTo(b) < 0;
    public static bool operator >(EntityId a, EntityId b) => a.CompareTo(b) > 0;
}
```

### Generic Constraints

```csharp
// Require sortable items
public class SortedBuffer<T> where T : IComparable<T>
{
    private List<T> _items = new();

    public void Add(T item)
    {
        int index = _items.BinarySearch(item);
        if (index < 0) index = ~index;
        _items.Insert(index, item);
    }
}

// Require items usable as dictionary keys
public class Registry<TKey, TValue> where TKey : IEquatable<TKey>
{
    private Dictionary<TKey, TValue> _data = new();
}
```

---

## Performance Considerations

### Interface Dispatch Cost

```csharp
// Virtual dispatch through interface
void ProcessViaInterface(IEnumerable<int> items)
{
    foreach (var item in items) { } // Interface call per iteration
}

// Direct call on concrete type
void ProcessDirect(List<int> items)
{
    foreach (var item in items) { } // Optimized, no interface dispatch
}
```

In hot paths (code running every frame), prefer concrete types. For API boundaries and flexibility, use interfaces.

### Boxing with Value Types

```csharp
struct Point : IEquatable<Point>
{
    public int X, Y;
    public bool Equals(Point other) => X == other.X && Y == other.Y;
}

Point a = new Point { X = 1, Y = 2 };
Point b = new Point { X = 1, Y = 2 };

// No boxing - calls IEquatable<Point>.Equals directly
bool eq1 = a.Equals(b);

// Boxing - Point converted to object
bool eq2 = ((object)a).Equals(b);

// Dictionary<Point, T> uses IEquatable<Point> - no boxing
// Hashtable uses object.Equals - boxes every lookup
```

### LINQ Allocation

```csharp
// Allocates iterator objects
var filtered = enemies.Where(e => e.IsActive).ToList();

// Zero allocation alternative for hot paths
for (int i = 0; i < enemies.Count; i++)
{
    if (enemies[i].IsActive)
        ProcessEnemy(enemies[i]);
}
```

---

## Unity-Specific Patterns

### Serialization-Friendly Collections

```csharp
// Unity can't serialize Dictionary - use parallel lists
[System.Serializable]
public class SerializableDictionary<TKey, TValue> : IEnumerable<KeyValuePair<TKey, TValue>>
{
    [SerializeField] private List<TKey> _keys = new();
    [SerializeField] private List<TValue> _values = new();

    private Dictionary<TKey, TValue> _cache;

    public TValue this[TKey key]
    {
        get => GetCache()[key];
        set
        {
            int index = _keys.IndexOf(key);
            if (index >= 0)
                _values[index] = value;
            else
            {
                _keys.Add(key);
                _values.Add(value);
            }
            _cache = null;
        }
    }

    private Dictionary<TKey, TValue> GetCache()
    {
        if (_cache == null)
        {
            _cache = new Dictionary<TKey, TValue>();
            for (int i = 0; i < _keys.Count; i++)
                _cache[_keys[i]] = _values[i];
        }
        return _cache;
    }

    public IEnumerator<KeyValuePair<TKey, TValue>> GetEnumerator() 
        => GetCache().GetEnumerator();
    
    IEnumerator IEnumerable.GetEnumerator() => GetEnumerator();
}
```

### Object Pooling with IDisposable

```csharp
public class PooledObject<T> : IDisposable where T : class, new()
{
    private static Stack<T> _pool = new();
    
    public T Value { get; private set; }

    public PooledObject()
    {
        Value = _pool.Count > 0 ? _pool.Pop() : new T();
    }

    public void Dispose()
    {
        _pool.Push(Value);
        Value = null;
    }
}

// Usage
using (var pooled = new PooledObject<List<int>>())
{
    pooled.Value.Add(1);
    pooled.Value.Add(2);
    // List returned to pool at end of scope
}
```

---

## Summary

**Essential for Unity:**

- `IEnumerator` — Coroutines
- `IEnumerable<T>` — LINQ, foreach, collections
- `IDisposable` — Native resource cleanup
- `IEquatable<T>` — Efficient dictionary/hashset keys

**Frequently Useful:**

- `IComparable<T>` — Sorting
- `IComparer<T>` — Multiple sort orders
- `IEqualityComparer<T>` — Custom equality rules

**Design Guidance:**

- Accept the most general interface that provides needed functionality
- Return concrete types when implementation details matter
- Use read-only interfaces to express immutability intent
- Implement `IEquatable<T>` on structs used as keys
- Implement `IDisposable` when holding unmanaged resources