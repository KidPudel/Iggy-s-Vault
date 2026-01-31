# Scene Management

> [!quote] The Core Insight Scenes are containers. Load them additively, not as replacements.

---

## What is a Scene?

A Scene (`.unity` file) is Unity's equivalent of a Godot scene (`.tscn`). It contains:

- GameObjects and their hierarchy
- Their components and serialized values
- Environment data (lighting, navigation bakes)

When you open Unity, you're editing a scene. When you press Play, Unity runs that scene.

---

## The Problem: What Happens When You Load a New Scene?

In Godot, when you call `change_scene_to_file()`, the old scene is freed and the new one loads. Simple.

Unity's default behavior is the same:

```csharp
SceneManager.LoadScene("Level02");
```

This **destroys everything** in the current scene and loads Level02. Your player? Gone. Your GameManager? Gone. Your score? Gone.

### The Old Solution: DontDestroyOnLoad

Unity developers needed a way to keep certain objects alive between scene loads. Unity's answer was `DontDestroyOnLoad`:

```csharp
public class GameManager : MonoBehaviour
{
    private void Awake()
    {
        DontDestroyOnLoad(gameObject);  // "Don't destroy me when scene changes"
    }
}
```

This moves the GameObject to a special hidden scene called "DontDestroyOnLoad" that persists across scene loads.

**Why this causes problems:**

- Objects accumulate in this hidden scene (hard to track)
- If you load a scene that also has a GameManager, now you have two
- You need extra singleton logic to prevent duplicates
- No clean way to "reset" the game without restarting
- Hard to debug — you can't see this hidden scene easily

---

## The Better Solution: Additive Scene Loading

Unity can load **multiple scenes at the same time**. This is the key insight.

```csharp
// Instead of replacing scenes...
SceneManager.LoadScene("Level02");

// ...load scenes alongside each other
SceneManager.LoadScene("Level02", LoadSceneMode.Additive);
```

With additive loading, nothing is destroyed. The new scene loads alongside existing scenes.

---

## The Pattern: Boot + Persistent + Content

Instead of using DontDestroyOnLoad, structure your game like this:

```
Boot.unity        → Tiny scene, just loads the others
Persistent.unity  → Managers that live forever (GameManager, AudioManager, etc.)
MainMenu.unity    → Main menu UI
Level01.unity     → Level content
Level02.unity     → Level content
```

### What is the Boot Scene?

The Boot scene is a pattern, not a Unity feature. It's a nearly-empty scene that exists only to load your other scenes. You create it yourself.

**Why have a Boot scene?**

Unity needs a "first scene" to run when the game starts. Instead of making MainMenu or Level01 the first scene (which creates complications), you create a tiny Boot scene whose only job is to set up your scene structure.

```csharp
// Boot.cs — the only script in Boot.unity
public class Boot : MonoBehaviour
{
    private void Start()
    {
        // Load the persistent managers
        SceneManager.LoadScene("Persistent", LoadSceneMode.Additive);
        
        // Load the main menu
        SceneManager.LoadScene("MainMenu", LoadSceneMode.Additive);
    }
}
```

After this runs, you have three scenes loaded:

- Boot (nearly empty, just did its job)
- Persistent (your managers)
- MainMenu (the actual menu)

### What is the Persistent Scene?

The Persistent scene is also a pattern you create. It holds all the GameObjects that should never be destroyed:

```
Persistent.unity contains:
├── GameManager
├── AudioManager  
├── InputManager
├── MusicPlayer
└── EventSystem (for UI)
```

**Why this replaces DontDestroyOnLoad:**

You never unload the Persistent scene. It just stays loaded forever. When you switch from MainMenu to Level01:

```csharp
public void StartGame()
{
    SceneManager.LoadScene("Level01", LoadSceneMode.Additive);
    SceneManager.UnloadSceneAsync("MainMenu");
}
```

Now you have:

- Boot (still there, doing nothing)
- Persistent (still there, managers intact)
- Level01 (just loaded)

Your managers survived because you never unloaded their scene.

---

## Setting the First Scene (Build Settings)

Unity needs to know which scene to load first when the game starts.

**File → Build Settings:**

```
Scenes In Build:
  ✓ Scenes/Boot.unity          0   ← Index 0 = loads first
  ✓ Scenes/Persistent.unity    1
  ✓ Scenes/MainMenu.unity      2
  ✓ Scenes/Level01.unity       3
```

The scene at index 0 is the entry point. Make it your Boot scene.

**Important:** Only scenes in Build Settings can be loaded at runtime. If you forget to add a scene here, `LoadScene` will fail.

---

## Loading and Unloading

### Synchronous (Freezes Game)

```csharp
SceneManager.LoadScene("Level01", LoadSceneMode.Additive);
```

The game freezes until the scene is fully loaded. Fine for tiny scenes, bad for large ones.

### Asynchronous (Smooth)

```csharp
await SceneManager.LoadSceneAsync("Level01", LoadSceneMode.Additive);
```

Loads in the background. The game keeps running.

### Unloading

```csharp
SceneManager.UnloadSceneAsync("Level01");
```

Destroys all GameObjects in that scene. Only works for additively-loaded scenes.

### Active Scene

When you have multiple scenes loaded, Unity needs to know which one is "active" — this determines where newly instantiated objects go.

```csharp
// Set Level01 as active (new Instantiate calls will put objects here)
SceneManager.SetActiveScene(SceneManager.GetSceneByName("Level01"));
```

---

## Typical Flow

```
Game starts
    └── Unity loads Boot.unity (index 0)
        └── Boot.cs Start() runs
            ├── Loads Persistent.unity (additive)
            └── Loads MainMenu.unity (additive)

Player clicks "Start Game"
    └── GameManager.StartGame()
        ├── Loads Level01.unity (additive)
        └── Unloads MainMenu.unity

Player beats level
    └── GameManager.LoadNextLevel()
        ├── Loads Level02.unity (additive)
        └── Unloads Level01.unity

Player quits to menu
    └── GameManager.ReturnToMenu()
        ├── Loads MainMenu.unity (additive)
        └── Unloads Level02.unity
```

Throughout all of this, Persistent.unity stays loaded. Your managers never die.

---

## API Quick Reference

```csharp
// Load
SceneManager.LoadScene("Name", LoadSceneMode.Additive);
await SceneManager.LoadSceneAsync("Name", LoadSceneMode.Additive);

// Unload
await SceneManager.UnloadSceneAsync("Name");

// Query
SceneManager.GetActiveScene()
SceneManager.GetSceneByName("Name")
SceneManager.sceneCount
scene.isLoaded
scene.GetRootGameObjects()

// Set active
SceneManager.SetActiveScene(scene);

// Move object to scene
SceneManager.MoveGameObjectToScene(obj, scene);
```

---

## Godot Comparison

|Godot|Unity|
|---|---|
|`change_scene_to_file()`|`LoadScene()` with Single mode (default)|
|Adding scene as child node|`LoadScene()` with Additive mode|
|Autoloads|Persistent scene pattern|
|Current scene|`SceneManager.GetActiveScene()`|

---

## Key Takeaways

1. **Default LoadScene destroys everything** — that's why DontDestroyOnLoad exists
2. **DontDestroyOnLoad has problems** — hidden scene, duplicates, hard to debug
3. **Additive loading is better** — multiple scenes coexist
4. **Boot scene** — a pattern (you create it) to set up your scene structure
5. **Persistent scene** — a pattern (you create it) to hold managers that never unload
6. **Build Settings** — scenes must be added here to load at runtime, index 0 is entry point