# Unity Gotchas for Godot Developers

> [!quote] The Core Insight Unity isn't Godot with different syntax. These specific differences will bite you.

---

## Start Runs Once, Ever

**Godot:** `_ready()` runs each time a node enters the tree.

**Unity:** `Start()` runs **once per lifetime**, even if you disable/enable the object.

```csharp
// WRONG expectation
private void Start()
{
    ResetState();  // Won't run again when re-enabled
}

// CORRECT
private void OnEnable()
{
    ResetState();  // Runs each time enabled
}
```

Use `OnEnable()` for "each time activated" logic.

---

## Fake Null

Destroyed objects don't become `null` immediately. Unity overloads `==` to fake it.

```csharp
Destroy(gameObject);

// Same frame:
if (gameObject == null)  // true (Unity's override)
    Debug.Log("Looks null");
    
if (gameObject is null)  // false (actual C# null check)
    Debug.Log("Actually null");  // Won't print

// The C# object still exists, just marked as destroyed
```

**Safe pattern:**

```csharp
if (gameObject)  // Implicit bool conversion — handles destroyed objects
{
    // Safe to use
}
```

---

## ScriptableObject Mutation Persists in Editor

**Godot:** Modifying a loaded Resource at runtime is common.

**Unity:** Modifying a ScriptableObject at runtime **persists after exiting Play Mode** in the Editor.

```csharp
[SerializeField] private ItemData _item;  // ScriptableObject

private void Start()
{
    _item.count = 99;  // BAD: Changes the asset permanently in Editor
}
```

**Fix:** Never modify ScriptableObjects. Use plain C# classes for mutable state.

See [[ScriptableObjects]] for the Data/State pattern.
See [[Data-Driven Design in Unity]] on more about saving

---

## No Built-in Signals

**Godot:** Every Object has signals. `signal health_changed` + `emit_signal()`.

**Unity:** No built-in signal system. You must use:

- C# events (`event Action`)
- UnityEvents (Inspector-configurable)
- ScriptableObject event channels

See [[Communication Patterns in Unity]] for options.

---

## Execution Order is Undefined

**Godot:** Children's `_ready()` runs before parent's. Tree order is mostly predictable.

**Unity:** Script execution order is **undefined** by default. Two objects' `Awake()` can run in any order.

```csharp
// Might fail if OtherManager.Awake() hasn't run yet
private void Awake()
{
    OtherManager.Instance.Register(this);  // Potential null
}
```

**Fix:** Use `[DefaultExecutionOrder]` or explicit initialization.

See [[Execution Order in Unity]].

---

## GetComponent Returns Null, Not Error

**Godot:** `get_node()` errors if path doesn't exist (unless using `get_node_or_null()`).

**Unity:** `GetComponent<T>()` returns `null` silently if component doesn't exist.

```csharp
var rb = GetComponent<Rigidbody>();
rb.AddForce(Vector3.up);  // NullReferenceException if no Rigidbody
```

**Safe pattern:**

```csharp
if (TryGetComponent<Rigidbody>(out var rb))
{
    rb.AddForce(Vector3.up);
}
```

Or use `[RequireComponent]`:

```csharp
[RequireComponent(typeof(Rigidbody))]
public class Movement : MonoBehaviour { }
```

---

## Transform is Separate from GameObject

**Godot:** Position is on the Node directly (`position`, `global_position`).

**Unity:** Position is on the Transform component.

```csharp
// WRONG
gameObject.position = Vector3.zero;  // Won't compile

// CORRECT
transform.position = Vector3.zero;
// or
gameObject.transform.position = Vector3.zero;
```

---

## Local vs World Confusion

**Godot:** `position` = local, `global_position` = world.

**Unity:** `transform.localPosition` = local, `transform.position` = world.

```csharp
// World position (where it actually is)
transform.position = new Vector3(10, 0, 0);

// Local position (relative to parent)
transform.localPosition = new Vector3(1, 0, 0);
```

If no parent, they're equal. With a parent, they differ.

---

## Instantiate Returns GameObject

**Godot:** `scene.instantiate()` returns the node type.

**Unity:** `Instantiate()` returns `GameObject` (or the type of what you passed in).

```csharp
// Returns GameObject
GameObject enemy = Instantiate(enemyPrefab);
var component = enemy.GetComponent<Enemy>();

// Better: reference the component type in the field
[SerializeField] private Enemy _enemyPrefab;  // Not GameObject
Enemy enemy = Instantiate(_enemyPrefab);  // Returns Enemy directly
```

---

## No $ Shorthand

**Godot:** `$Child` gets a child node.

**Unity:** No shorthand. Use `[SerializeField]` or `transform.Find()`.

```csharp
// Assign in Inspector (preferred)
[SerializeField] private Transform _child;

// Or find by name (fragile)
Transform child = transform.Find("ChildName");
```

---

## Rotation is Quaternion

**Godot:** Rotation is typically Euler angles (degrees).

**Unity:** Internal rotation is `Quaternion`. Euler angles are converted.

```csharp
// Quaternion (internal)
transform.rotation = Quaternion.identity;
transform.rotation = Quaternion.Euler(0, 90, 0);

// Euler angles (converted, can have issues)
transform.eulerAngles = new Vector3(0, 90, 0);

// Rotating
transform.Rotate(Vector3.up, 45f);
```

Quaternion math avoids gimbal lock. Learn `Quaternion.Lerp`, `Quaternion.LookRotation`, etc.

---

## No queue_free, Use Destroy

**Godot:** `queue_free()` removes at end of frame.

**Unity:** `Destroy(gameObject)` removes at end of frame.

```csharp
Destroy(gameObject);        // Destroys at end of frame
Destroy(gameObject, 2f);    // Destroys after 2 seconds
DestroyImmediate(gameObject);  // Destroys NOW (avoid in runtime)
```

> [!warning] Don't Use DestroyImmediate in Game Code It's for Editor scripts only. Use `Destroy()` at runtime.

---

## Arrays Start at 0 (Same as Godot)

This isn't a gotcha — just confirming C# arrays are 0-indexed like GDScript.

---

## Coroutines vs await

**Godot:** `await` with signals and timers.

**Unity:** Coroutines with `yield return`, or async/await (Unity 2023+).

```csharp
// Coroutine
IEnumerator DoThing()
{
    yield return new WaitForSeconds(1f);
    DoSomething();
}
StartCoroutine(DoThing());

// Async (Unity 2023+)
async void DoThingAsync()
{
    await Task.Delay(1000);
    DoSomething();
}
```

---

## No @export_category, Use Attributes

**Godot:** `@export_group`, `@export_category`.

**Unity:** Attributes for organization.

```csharp
[Header("Movement")]
[SerializeField] private float _speed;

[Space(10)]

[Header("Combat")]
[SerializeField] private int _damage;

[Tooltip("Seconds between attacks")]
[SerializeField] private float _cooldown;
```

---

## C# Naming Conventions

**Godot/GDScript:** `snake_case` for variables and functions.

**Unity/C#:** Different conventions.

```csharp
public class PlayerController : MonoBehaviour  // PascalCase class
{
    [SerializeField] private float _moveSpeed;  // _camelCase private field
    
    public float MoveSpeed => _moveSpeed;       // PascalCase property
    
    public void TakeDamage(int amount) { }      // PascalCase method
    
    private void handleInput() { }              // camelCase private method (some styles)
}
```

---

## Summary Table

| Godot                 | Unity                          | Gotcha                        |
| --------------------- | ------------------------------ | ----------------------------- |
| `_ready()`            | `Start()`                      | Start runs once ever          |
| `is_instance_valid()` | `if (obj)`                     | Fake null                     |
| Modify Resource       | DON'T modify SO                | Persists in Editor            |
| `signal`              | `event Action`                 | No built-in signals           |
| Tree order            | Undefined order                | Use `[DefaultExecutionOrder]` |
| `get_node()`          | `GetComponent()`               | Returns null, not error       |
| `position`            | `transform.position`           | Transform is separate         |
| `$Child`              | `[SerializeField]` or `Find()` | No shorthand                  |
| Euler rotation        | Quaternion                     | Learn Quaternion API          |
| `queue_free()`        | `Destroy()`                    | Same concept                  |
| `await signal`        | Coroutines / async             | Different syntax              |