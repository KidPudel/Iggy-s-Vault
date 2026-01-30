# Unity Day 0: Setup Guide for Godot Developers

> You know game dev. You know architecture. Here's where everything lives in Unity.

---

## Creating a Project

**Unity Hub → New Project**

|Template|Use For|
|---|---|
|3D (Built-in)|Simple 3D, learning|
|3D (URP)|Modern 3D with good performance|
|3D (HDRP)|High-end visuals (PC/console)|
|2D (Built-in)|Simple 2D|
|2D (URP)|Modern 2D with lighting|

> **Recommendation:** Use URP for new projects. Built-in is legacy.

---

## The Editor Layout

```
┌─────────────────────────────────────────────────────────────────────────┐
│  Toolbar   [Play] [Pause] [Step]                                        │
├──────────────────┬──────────────────────────┬───────────────────────────┤
│                  │                          │                           │
│    Hierarchy     │      Scene / Game        │       Inspector           │
│                  │                          │                           │
│  (Scene tree)    │   (Visual editing /      │   (Selected object's      │
│                  │    preview)              │    components)            │
│                  │                          │                           │
├──────────────────┴──────────────────────────┴───────────────────────────┤
│                           Project                                       │
│                     (Files / Assets)                                    │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

|Panel|Godot Equivalent|Purpose|
|---|---|---|
|**Hierarchy**|Scene tree dock|Objects in current scene|
|**Scene**|2D/3D viewport|Visual editing|
|**Game**|Running game|Preview (only active in Play)|
|**Inspector**|Inspector dock|Properties of selected object|
|**Project**|FileSystem dock|All assets in project|
|**Console**|Output|Logs, errors, warnings|

**Open Console:** Window → General → Console (or press `` ` `` )

---

## Essential Settings (Do This First)

### 1. Set Up Folders

In Project panel, create:

```
Assets/
├── _Project/
│   ├── Scenes/
│   ├── Scripts/
│   ├── Prefabs/
│   ├── Art/
│   └── Data/
```

### 2. Install Essential Packages

Window → Package Manager → Unity Registry

- [x] **TextMeshPro** (may prompt on first use)
- [x] **Input System**
- [x] **Cinemachine** (if 3D)

When prompted about Input System: select "Both" to keep legacy input during learning.

### 3. Editor Preferences

Edit → Preferences:

- **External Tools** → Set your IDE (VS Code, Rider, Visual Studio)
- **Colors** → Playmode tint (helps know when you're in Play mode)

### 4. Project Settings

Edit → Project Settings:

|Setting|Location|What to Set|
|---|---|---|
|Company/Product Name|Player|Your info|
|Default Scene|—|Set via Build Settings|
|Fixed Timestep|Time|0.02 (default, 50 fps physics)|
|Gravity|Physics|-9.81 Y default|

---

## Scene Setup

### Creating a Scene

1. Right-click in Project → Create → Scene
2. Name it (e.g., `Main.unity`)
3. Double-click to open

### Scene Contains

When you create a scene, it starts with:

- **Main Camera** — required to see anything
- **Directional Light** — basic lighting

### Adding Objects

**Right-click in Hierarchy:**

|Menu Item|Creates|
|---|---|
|Create Empty|Empty GameObject (like Node)|
|3D Object → Cube/Sphere/etc.|Primitives|
|2D Object → Sprite|2D sprite|
|UI → Canvas|UI root (required for UI)|
|Light → Point Light|Light source|

### Organizing the Hierarchy

Create empty GameObjects as folders:

```
Hierarchy:
├── --- MANAGERS ---      (empty, just for organization)
│   ├── GameManager
│   └── AudioManager
├── --- ENVIRONMENT ---
│   ├── Ground
│   └── Walls
├── --- ENTITIES ---
│   ├── Player
│   └── Enemies
└── --- UI ---
    └── Canvas
```

> **Tip:** Start names with `---` or use brackets `[MANAGERS]` to visually separate sections.

---

## Setting the Startup Scene

**File → Build Settings** (Ctrl+Shift+B)

1. Click **Add Open Scenes** (or drag scenes from Project)
2. **First scene (index 0) is the startup scene**
3. Drag to reorder

```
Scenes In Build:
  ✓ Scenes/Boot.unity          0   ← This runs first
  ✓ Scenes/MainMenu.unity      1
  ✓ Scenes/Level01.unity       2
```

---

## Creating Your First Script

### Create Script

Right-click in Project → Create → C# Script → Name it `PlayerController`

> **Important:** Name the file correctly. Renaming later causes issues.

### Basic Script Template

```csharp
using UnityEngine;

public class PlayerController : MonoBehaviour
{
    [SerializeField] private float _speed = 5f;
    
    private void Awake()
    {
        // Self-initialization (like _init + _enter_tree)
    }
    
    private void Start()
    {
        // Cross-object setup (like _ready, but runs ONCE ever)
    }
    
    private void Update()
    {
        // Every frame (like _process)
    }
    
    private void FixedUpdate()
    {
        // Physics tick (like _physics_process)
    }
}
```

### Attach Script to GameObject

1. Select GameObject in Hierarchy
2. Drag script to Inspector, **OR**
3. Inspector → Add Component → Search script name

---

## Creating a Prefab

1. Set up a GameObject in the scene (add components, children, etc.)
2. Drag it from **Hierarchy** into **Project** folder
3. It's now a Prefab (blue icon in Hierarchy)

**To instantiate:**

```csharp
[SerializeField] private GameObject _enemyPrefab;

void SpawnEnemy()
{
    Instantiate(_enemyPrefab, transform.position, Quaternion.identity);
}
```

---

## Play Mode

|Button|Action|
|---|---|
|▶ Play|Enter play mode|
|⏸ Pause|Pause game|
|⏭ Step|Advance one frame|

> ⚠️ **Changes in Play Mode are NOT saved.** When you exit, everything reverts. This catches everyone at first.

**Tip:** Edit → Preferences → Colors → Playmode tint. Set a visible color so you know when you're in Play mode.

---

## Inspector Basics

### Assigning References

```csharp
[SerializeField] private Transform _target;
[SerializeField] private GameObject _prefab;
[SerializeField] private AudioClip _sound;
```

Then drag objects/assets to the fields in Inspector.

### Exposed Fields

|Code|Shows In Inspector|
|---|---|
|`public int health;`|Yes|
|`private int _health;`|No|
|`[SerializeField] private int _health;`|Yes (preferred)|
|`[HideInInspector] public int health;`|No|

---

## Input (Quick Start)

### Legacy Input (for prototyping)

```csharp
void Update()
{
    float h = Input.GetAxis("Horizontal");  // A/D or Left/Right
    float v = Input.GetAxis("Vertical");    // W/S or Up/Down
    
    if (Input.GetKeyDown(KeyCode.Space))
        Jump();
    
    if (Input.GetMouseButtonDown(0))  // Left click
        Fire();
}
```

### New Input System (for production)

1. Create Input Actions asset (right-click → Create → Input Actions)
2. Define actions (Move, Jump, Fire)
3. Add **PlayerInput** component to player
4. Receive via methods:

```csharp
public void OnMove(InputValue value)
{
    Vector2 input = value.Get<Vector2>();
}

public void OnJump(InputValue value)
{
    if (value.isPressed) Jump();
}
```

---

## Common Tasks Quick Reference

|Task|How|
|---|---|
|Create empty object|Hierarchy → Right-click → Create Empty|
|Add component|Inspector → Add Component|
|Create script|Project → Right-click → Create → C# Script|
|Create material|Project → Right-click → Create → Material|
|Create prefab|Drag from Hierarchy to Project|
|Edit prefab|Double-click prefab in Project|
|Find asset|Ctrl+P (or Project search bar)|
|Find in scene|Hierarchy search bar|
|Duplicate|Ctrl+D|
|Delete|Delete key|
|Focus on object|Select + F|
|Frame all|Hierarchy → Select all → F|

---

## Keyboard Shortcuts

|Shortcut|Action|
|---|---|
|**Ctrl+S**|Save scene|
|**Ctrl+P**|Play/Stop|
|**Ctrl+Shift+P**|Pause|
|**Ctrl+D**|Duplicate|
|**Ctrl+Z**|Undo|
|**F**|Focus selected|
|**W**|Move tool|
|**E**|Rotate tool|
|**R**|Scale tool|
|**Q**|View tool (pan)|
|**Alt+Click drag**|Orbit in Scene view|
|**Middle mouse drag**|Pan in Scene view|
|**Scroll**|Zoom|

---

## Where Things Are (Godot → Unity)

|Godot|Unity Location|
|---|---|
|Project Settings|Edit → Project Settings|
|Editor Settings|Edit → Preferences|
|Export|File → Build Settings|
|Script Editor setting|Preferences → External Tools|
|Input Map|Project Settings → Input Manager (legacy) or Input Actions asset|
|Autoloads|Use persistent scene pattern|
|FileSystem dock|Project panel|
|Scene tree|Hierarchy panel|
|Node properties|Inspector panel|
|Animation editor|Window → Animation → Animation|
|Tilemap editor|Create Tilemap in Hierarchy|
|Debugger|IDE debugger + Console window|

---

## Build Your Game

**File → Build Settings:**

1. Add scenes
2. Select platform (PC, Mac, Linux, etc.)
3. Click **Switch Platform** if needed
4. **Player Settings** for icons, resolution, etc.
5. **Build** or **Build and Run**

---

## Recommended First Steps

### Day 0 Checklist

1. [ ] Create new URP project
2. [ ] Set up folder structure
3. [ ] Install Input System, TextMeshPro, Cinemachine
4. [ ] Create main scene
5. [ ] Add Player (3D: Capsule + Rigidbody, 2D: Sprite + Rigidbody2D)
6. [ ] Create PlayerController script
7. [ ] Add basic movement
8. [ ] Save scene
9. [ ] Set as startup scene in Build Settings
10. [ ] Press Play and move around

### Minimal Player (3D)

```csharp
using UnityEngine;

public class PlayerController : MonoBehaviour
{
    [SerializeField] private float _speed = 5f;
    
    private Rigidbody _rb;
    
    private void Awake()
    {
        _rb = GetComponent<Rigidbody>();
    }
    
    private void FixedUpdate()
    {
        float h = Input.GetAxis("Horizontal");
        float v = Input.GetAxis("Vertical");
        
        Vector3 movement = new Vector3(h, 0, v) * _speed;
        _rb.linearVelocity = new Vector3(movement.x, _rb.linearVelocity.y, movement.z);
    }
}
```

Setup:

1. Create → 3D Object → Capsule
2. Add Component → Rigidbody
3. Freeze Rotation X, Z in Rigidbody constraints
4. Attach script
5. Play

### Minimal Player (2D)

```csharp
using UnityEngine;

public class PlayerController2D : MonoBehaviour
{
    [SerializeField] private float _speed = 5f;
    
    private Rigidbody2D _rb;
    
    private void Awake()
    {
        _rb = GetComponent<Rigidbody2D>();
    }
    
    private void FixedUpdate()
    {
        float h = Input.GetAxis("Horizontal");
        float v = Input.GetAxis("Vertical");
        
        _rb.linearVelocity = new Vector2(h, v) * _speed;
    }
}
```

Setup:

1. Create → 2D Object → Sprite → Square (or add sprite)
2. Add Component → Rigidbody2D
3. Set Gravity Scale to 0 (for top-down) or leave for platformer
4. Add BoxCollider2D
5. Attach script
6. Play

---

## You're Ready

You now know:

- Where everything is
- How to create scenes and set startup
- How to create scripts and prefabs
- Basic input handling
- How to build

Go make something. Refer to the other docs when you need depth.