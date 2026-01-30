# Scene Management

> [!quote] The Core Insight Scenes are containers. Load them additively, not as replacements.

---

## What Scenes Are

A Scene is a file (`.unity`) containing:

- GameObjects and their hierarchy
- Their components and serialized values
- Lighting and navigation data

Scenes are like Godot's main scenes, but Unity can have **multiple scenes loaded simultaneously**.

---

## Loading Modes

### Single (Default) — Avoid

```csharp
SceneManager.LoadScene("Level01");
// or
SceneManager.LoadScene(1);  // Build index
```

**Unloads everything**, then loads the new scene. Objects marked `DontDestroyOnLoad` survive.

**Problems:**

- No control over what persists
- Can't keep managers alive cleanly
- Scene transitions feel abrupt

### Additive — Recommended

```csharp
SceneManager.LoadScene("Level01", LoadSceneMode.Additive);
```

**Loads alongside existing scenes.** Nothing is unloaded.

```
Before:                      After:
├── PersistentScene          ├── PersistentScene
                             └── Level01
```

---

## Recommended Architecture

```
Scenes/
├── Boot.unity           # Entry point, loads Persistent
├── Persistent.unity     # Managers, always loaded
├── MainMenu.unity       # UI scene
├── Gameplay.unity       # Core gameplay systems
├── Level01.unity        # Level content
└── Level02.unity        # Level content
```

### Boot Scene

Loads first (set in Build Settings), then loads Persistent:

```csharp
public class Boot : MonoBehaviour
{
    private void Start()
    {
        SceneManager.LoadScene("Persistent", LoadSceneMode.Additive);
    }
}
```

### Persistent Scene

Contains managers that live forever:

```
Persistent (Scene)
├── GameManager
├── AudioManager
├── InputManager
└── UIManager
```

No `DontDestroyOnLoad` needed — this scene just never unloads.

---

## Loading and Unloading

### Async Loading (Recommended)

```csharp
public class LevelLoader : MonoBehaviour
{
    private string _currentLevel;
    
    public async void LoadLevel(string levelName)
    {
        // Unload current
        if (!string.IsNullOrEmpty(_currentLevel))
        {
            await SceneManager.UnloadSceneAsync(_currentLevel);
        }
        
        // Load new
        await SceneManager.LoadSceneAsync(levelName, LoadSceneMode.Additive);
        _currentLevel = levelName;
        
        // Optional: set as active scene (affects new object creation)
        SceneManager.SetActiveScene(SceneManager.GetSceneByName(levelName));
    }
}
```

### With Loading Screen

```csharp
public async void LoadLevelWithProgress(string levelName)
{
    _loadingScreen.Show();
    
    var operation = SceneManager.LoadSceneAsync(levelName, LoadSceneMode.Additive);
    operation.allowSceneActivation = false;  // Don't activate yet
    
    while (operation.progress < 0.9f)
    {
        _loadingScreen.SetProgress(operation.progress);
        await Task.Yield();
    }
    
    operation.allowSceneActivation = true;
    await Task.Yield();  // Wait for activation
    
    _loadingScreen.Hide();
}
```

---

## DontDestroyOnLoad — Avoid

```csharp
// Old pattern — not recommended
private void Awake()
{
    DontDestroyOnLoad(gameObject);
}
```

**Problems:**

- Objects pile up in a hidden "DontDestroyOnLoad" scene
- No clear ownership
- Hard to debug
- Lose control of object lifetime

**Better alternative:** Keep persistent objects in a dedicated scene that never unloads.

---

## Scene Queries

```csharp
// Get loaded scenes
int sceneCount = SceneManager.sceneCount;
Scene scene = SceneManager.GetSceneAt(0);

// Get scene by name
Scene scene = SceneManager.GetSceneByName("Level01");

// Check if loaded
bool isLoaded = scene.isLoaded;

// Get active scene (where new objects spawn)
Scene active = SceneManager.GetActiveScene();

// Get root objects of a scene
GameObject[] roots = scene.GetRootGameObjects();
```

---

## Moving Objects Between Scenes

```csharp
// Move to specific scene
SceneManager.MoveGameObjectToScene(gameObject, targetScene);

// Move to active scene
SceneManager.MoveGameObjectToScene(gameObject, SceneManager.GetActiveScene());
```

---

## Build Settings

Scenes must be in Build Settings to load by name or index.

File → Build Settings → Add Open Scenes (or drag scenes in)

```csharp
// Load by build index
SceneManager.LoadScene(0);  // First scene in build settings

// Get current scene's build index
int index = SceneManager.GetActiveScene().buildIndex;
```

---

## Common Patterns

### Scene-Based State

```csharp
public class GameManager : MonoBehaviour
{
    public async void StartGame()
    {
        await SceneManager.LoadSceneAsync("Gameplay", LoadSceneMode.Additive);
        await SceneManager.LoadSceneAsync("Level01", LoadSceneMode.Additive);
        SceneManager.UnloadSceneAsync("MainMenu");
    }
    
    public async void ReturnToMenu()
    {
        await SceneManager.LoadSceneAsync("MainMenu", LoadSceneMode.Additive);
        SceneManager.UnloadSceneAsync("Gameplay");
        SceneManager.UnloadSceneAsync(_currentLevel);
    }
}
```

### Scene Initialization

```csharp
public class LevelInitializer : MonoBehaviour
{
    [SerializeField] private Transform _playerSpawn;
    
    private void Start()
    {
        // Find player from Persistent scene
        var player = FindObjectOfType<Player>();
        player.transform.position = _playerSpawn.position;
    }
}
```

---

## Godot Comparison

|Godot|Unity|
|---|---|
|`get_tree().change_scene_to_file()`|`SceneManager.LoadScene()` (Single)|
|Adding child scene to tree|`LoadScene(_, LoadSceneMode.Additive)`|
|`get_tree().current_scene`|`SceneManager.GetActiveScene()`|
|`get_tree().reload_current_scene()`|`SceneManager.LoadScene(currentScene)`|
|Autoloads|Persistent scene (never unloaded)|

---

## Key Takeaways

1. **Use Additive loading** — don't replace scenes, layer them
2. **Persistent scene** — for managers, instead of DontDestroyOnLoad
3. **Async loading** — smoother transitions, progress tracking
4. **Unload explicitly** — additive scenes stay until you remove them
5. **Scenes must be in Build Settings** to load at runtime