# Transform & Hierarchy

> [!quote] The Core Insight Transform is both spatial data AND the scene tree structure.

---

## What Transform Does

Every GameObject has exactly one Transform. It handles two things:

1. **Spatial data** — position, rotation, scale
2. **Hierarchy** — parent-child relationships

You cannot remove Transform. You cannot add a second one.

---

## Spatial Data

### Position

```csharp
// World space (absolute)
transform.position = new Vector3(10, 5, 0);
Vector3 worldPos = transform.position;

// Local space (relative to parent)
transform.localPosition = new Vector3(1, 0, 0);  // 1 unit from parent
Vector3 localPos = transform.localPosition;
```

**World position** = where it actually is in the scene. **Local position** = offset from parent. If no parent, local == world.

### Rotation

```csharp
// Quaternion (internal representation)
transform.rotation = Quaternion.identity;
transform.localRotation = Quaternion.Euler(0, 90, 0);

// Euler angles (human readable, but beware gimbal lock)
Vector3 euler = transform.eulerAngles;
transform.eulerAngles = new Vector3(0, 90, 0);

// Rotating
transform.Rotate(Vector3.up, 45f);  // Rotate 45° around Y
transform.Rotate(Vector3.up, speed * Time.deltaTime);  // Per frame
```

### Scale

```csharp
// Local scale only (no world scale property)
transform.localScale = new Vector3(2, 2, 2);  // Double size

// Lossy scale (approximate world scale, read-only)
Vector3 approxWorld = transform.lossyScale;
```

> [!warning] Scale is Local Only There's no `transform.scale`. Use `localScale`. Non-uniform parent scaling can cause skewing.

---

## Hierarchy

### Parent-Child Relationships

```csharp
// Set parent
transform.SetParent(parentTransform);
transform.SetParent(parentTransform, worldPositionStays: true);  // Keep world position

// Get parent
Transform parent = transform.parent;

// Unparent (move to root)
transform.SetParent(null);
```

When a child moves with its parent:

- Child's **localPosition** stays the same
- Child's **world position** changes

### Finding Children

```csharp
// By name (immediate children only)
Transform child = transform.Find("ChildName");
Transform nested = transform.Find("Child/GrandChild");  // Path syntax

// By index
Transform first = transform.GetChild(0);
int count = transform.childCount;

// Iterate all children
foreach (Transform child in transform)
{
    Debug.Log(child.name);
}
```

> [!warning] Prefer Serialized References `transform.Find()` is fragile — rename breaks it. Use `[SerializeField]` when possible.

```csharp
// Better: assign in Inspector
[SerializeField] private Transform _weaponSlot;
```

### Hierarchy Queries

```csharp
// Root of hierarchy
Transform root = transform.root;

// Is child of?
bool isChild = transform.IsChildOf(otherTransform);

// Get all children (including nested)
var allChildren = GetComponentsInChildren<Transform>();
```

---

## Local vs World Space

```
World Space                     Local Space
     Y                              Y
     │                              │  (relative to parent)
     │                              │
     └───── X                       └───── X
    /                              /
   Z                              Z

transform.position              transform.localPosition
transform.rotation              transform.localRotation
transform.eulerAngles           transform.localEulerAngles
(no world scale)                transform.localScale
```

### Converting Between Spaces

```csharp
// Local → World
Vector3 worldPoint = transform.TransformPoint(localPoint);
Vector3 worldDir = transform.TransformDirection(localDir);

// World → Local
Vector3 localPoint = transform.InverseTransformPoint(worldPoint);
Vector3 localDir = transform.InverseTransformDirection(worldDir);
```

**Use case:** Spawning a projectile in front of the player.

```csharp
// "1 unit forward in my local space" → world position
Vector3 spawnPos = transform.TransformPoint(Vector3.forward);
```

---

## Direction Vectors

```csharp
// Local axes in world space
Vector3 forward = transform.forward;   // Local Z+ in world
Vector3 right = transform.right;       // Local X+ in world  
Vector3 up = transform.up;             // Local Y+ in world
```

These are **world-space directions** that represent where the object's local axes point.

```csharp
// Move forward (in the direction the object faces)
transform.position += transform.forward * speed * Time.deltaTime;
```

---

## Looking At Things

```csharp
// Face a target
transform.LookAt(targetTransform);
transform.LookAt(targetPosition);

// Face a direction
transform.forward = directionVector;
```

---

## Common Patterns

### Parenting on Spawn

```csharp
var bullet = Instantiate(bulletPrefab);
bullet.transform.SetParent(bulletsContainer, worldPositionStays: true);
```

### Keeping World Position When Parenting

```csharp
// worldPositionStays: true (default) — keeps world position, recalculates local
// worldPositionStays: false — keeps local position, world position changes

transform.SetParent(newParent, worldPositionStays: true);
```

### Resetting Local Transform

```csharp
transform.localPosition = Vector3.zero;
transform.localRotation = Quaternion.identity;
transform.localScale = Vector3.one;
```

---

## Hierarchy as Scene Structure

The hierarchy serves the same role as Godot's scene tree:

```
Scene
├── Managers
│   ├── GameManager
│   └── AudioManager
├── Environment
│   ├── Ground
│   └── Walls
├── Player
│   ├── Model
│   ├── Camera
│   └── WeaponSlot
└── Enemies
    ├── Goblin
    └── Orc
```

**Organizational GameObjects** (empty, just Transform) are common for grouping.

---

## Godot Comparison

|Godot|Unity|
|---|---|
|`position`|`transform.localPosition`|
|`global_position`|`transform.position`|
|`rotation`|`transform.localEulerAngles` (degrees)|
|`global_rotation`|`transform.eulerAngles`|
|`get_parent()`|`transform.parent`|
|`get_children()`|Iterate `transform` or `GetChild(i)`|
|`add_child(node)`|`child.transform.SetParent(transform)`|
|`get_node("Path")`|`transform.Find("Path")`|
|Scene tree|Hierarchy (same concept)|

---

## Key Takeaways

1. **Transform = position + rotation + scale + hierarchy**
2. **Local** = relative to parent, **World** = absolute
3. **Use localPosition** for most manipulation
4. **SetParent** controls hierarchy
5. **Prefer [SerializeField]** over Find()
6. **transform.forward/right/up** = local axes in world space