# Prefabs

> [!quote] The Core Insight A Prefab is a saved GameObject. Change the Prefab, all instances update.

---

## What Prefabs Are

A Prefab is a **reusable template** stored as an asset. It contains:

- A GameObject structure (with children)
- All components and their values
- References to other assets (materials, meshes, etc.)

```
Asset (on disk)              Instance (in scene)
Enemy.prefab        ──►      Enemy (Clone)
                             Enemy (Clone)
                             Enemy (Clone)
```

Similar to Godot's scene `.tscn` files, but with a key difference: Prefab instances maintain a **live link** to the source asset.

---

## Creating Prefabs

**From scene to asset:**

1. Set up a GameObject in the scene
2. Drag it into the Project window (Assets folder)
3. The scene object becomes an instance of the new Prefab

**Or right-click in Project:** Create → Prefab → blank Prefab, then edit.

---

## Instantiating

```csharp
// Basic spawn
GameObject enemy = Instantiate(enemyPrefab);

// With position and rotation
GameObject enemy = Instantiate(enemyPrefab, spawnPoint, Quaternion.identity);

// With parent
GameObject enemy = Instantiate(enemyPrefab, parentTransform);

// Full control
GameObject enemy = Instantiate(
    enemyPrefab, 
    position, 
    rotation, 
    parentTransform
);
```

### Getting the Prefab Reference

```csharp
public class Spawner : MonoBehaviour
{
    [SerializeField] private GameObject _enemyPrefab;  // Assign in Inspector
    
    public void Spawn()
    {
        Instantiate(_enemyPrefab, transform.position, Quaternion.identity);
    }
}
```

> [!warning] Prefab Reference vs Instance.
> The `[SerializeField]` field holds the **asset** reference, not a scene instance. Instantiate creates a **new instance** in the scene.

`[SerializeField] private GameObject _enemyPrefab;` is like holding a `PackedScene` reference — it's the template asset, not a live object. You call `Instantiate(_enemyPrefab)` to create an actual instance in the scene, just like `packed_scene.instantiate()` in Godot.


---

## Prefab Instances

An instance in the scene is linked to its Prefab asset.

**Blue text** in Hierarchy = Prefab instance.

### Overrides

Instances can override Prefab values:

- Change a field → **bold** in Inspector
- Add a component → shows as override
- Remove a component → shows as override
- Add/remove children → shows as override

```
Prefab: Enemy
├── Health: 100
└── Speed: 5

Instance in scene:
├── Health: 150    ← Override (bold)
└── Speed: 5       ← From Prefab
```

### Applying and Reverting

- **Apply** = push instance changes back to Prefab (affects all instances)
- **Revert** = discard instance changes, restore Prefab values

Right-click on component or use Overrides dropdown in Inspector.

---

## Prefab Mode

Double-click a Prefab asset to edit in **Prefab Mode** (isolated view).

Changes here affect the Prefab asset directly → all instances update.

---

## Nested Prefabs

Prefabs can contain other Prefabs:

```
Vehicle.prefab
├── Body (regular GameObject)
├── Wheel.prefab (nested)
├── Wheel.prefab (nested)
├── Wheel.prefab (nested)
└── Wheel.prefab (nested)
```

Update `Wheel.prefab` → all vehicles get new wheels.

---

## Prefab Variants

A variant **inherits** from a base Prefab:

```
Enemy.prefab (base)
├── EnemyGoblin.prefab (variant) — overrides: Health = 50, Speed = 7
└── EnemyOrc.prefab (variant) — overrides: Health = 200, Speed = 3
```

Create: Right-click Prefab → Create → Prefab Variant

Change the base → variants inherit changes (unless overridden).

---

## Instantiate with Type

If the Prefab root has a specific component, you can get it directly:

```csharp
[SerializeField] private Enemy _enemyPrefab;  // Reference the component type

void Spawn()
{
    Enemy enemy = Instantiate(_enemyPrefab);  // Returns Enemy, not GameObject
    enemy.Initialize(100);
}
```

This saves a `GetComponent` call.

---

## Destroying Instances

```csharp
// Destroy the GameObject (and all components/children)
Destroy(gameObject);

// Delayed destruction
Destroy(gameObject, 2f);  // After 2 seconds
```

Destroying an instance does not affect the Prefab or other instances.

---

## Common Patterns

### Object Pool Setup

```csharp
[SerializeField] private GameObject _bulletPrefab;
[SerializeField] private int _poolSize = 20;

private Queue<GameObject> _pool = new();

private void Awake()
{
    for (int i = 0; i < _poolSize; i++)
    {
        var bullet = Instantiate(_bulletPrefab);
        bullet.SetActive(false);
        _pool.Enqueue(bullet);
    }
}

public GameObject Get()
{
    var bullet = _pool.Dequeue();
    bullet.SetActive(true);
    return bullet;
}

public void Return(GameObject bullet)
{
    bullet.SetActive(false);
    _pool.Enqueue(bullet);
}
```

### Spawn with Configuration

```csharp
public Enemy SpawnEnemy(Vector3 position, EnemyData data)
{
    var enemy = Instantiate(_enemyPrefab, position, Quaternion.identity);
    enemy.Initialize(data);  // Configure after spawn
    return enemy;
}
```

---

## Godot Comparison

|Godot|Unity|
|---|---|
|`.tscn` (PackedScene)|Prefab (`.prefab`)|
|`scene.instantiate()`|`Instantiate(prefab)`|
|Scene inheritance|Prefab Variants|
|No live link after instantiation|Instances linked to Prefab|
|Editable as separate scene|Prefab Mode|

**Key difference:** Unity Prefab instances stay connected. Godot scene instances are independent after instantiation.

---

## When to Use Prefabs

**Use Prefabs for:**

- Anything you spawn at runtime (enemies, bullets, effects)
- Repeated objects in scenes (trees, props)
- Objects that need consistent updates (change Prefab → all update)

**Don't need Prefabs for:**

- One-off objects unique to a scene
- Objects built entirely from code

---

## Key Takeaways

1. **Prefab = saved GameObject template**
2. **Instances link to Prefab** — changes propagate
3. **Overrides** let instances differ from Prefab
4. **Variants** inherit from base Prefabs
5. **Instantiate** creates new instances at runtime
6. **[SerializeField]** to reference Prefabs in scripts