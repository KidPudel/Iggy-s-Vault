# Unity Static Classes & Singletons

> Built-in global access points. No setup needed — just use them.

---

TODO: add explanation and tips, but insightful and not cluttering

## Core

### Application

```csharp
// Paths
Application.persistentDataPath   // Save files here (survives updates)
Application.streamingAssetsPath  // Read-only data files
Application.dataPath             // Project Assets folder

// State
Application.isPlaying            // In play mode?
Application.isFocused            // Window focused?
Application.targetFrameRate = 60;

// Platform
Application.platform             // RuntimePlatform enum
Application.isMobilePlatform

// Control
Application.Quit();
Application.OpenURL("https://...");
```

### Debug

```csharp
Debug.Log("Info");
Debug.LogWarning("Warning");
Debug.LogError("Error");
Debug.LogException(exception);

// Conditional (stripped from release builds)
Debug.Assert(condition, "Failed!");

// Visual debugging (Scene view only)
Debug.DrawLine(start, end, Color.red, duration);
Debug.DrawRay(origin, direction, Color.green);
```

### Time

```csharp
Time.deltaTime          // Seconds since last frame (use in Update)
Time.fixedDeltaTime     // Fixed timestep (use in FixedUpdate)
Time.unscaledDeltaTime  // Ignores timeScale (for pause menus)
Time.time               // Seconds since game start
Time.timeScale          // 0 = paused, 1 = normal, 0.5 = slow-mo
Time.frameCount         // Total frames rendered
```

---

## Input

### Input (Legacy)

```csharp
// Axes (-1 to 1)
Input.GetAxis("Horizontal")      // Smoothed
Input.GetAxisRaw("Vertical")     // Raw: -1, 0, or 1

// Buttons (defined in Input Manager)
Input.GetButtonDown("Jump")      // Frame pressed
Input.GetButton("Fire1")         // Held
Input.GetButtonUp("Jump")        // Frame released

// Keys
Input.GetKeyDown(KeyCode.Space)
Input.GetKey(KeyCode.W)
Input.GetKeyUp(KeyCode.Escape)

// Mouse
Input.mousePosition              // Screen coordinates
Input.GetMouseButtonDown(0)      // 0=left, 1=right, 2=middle
Input.mouseScrollDelta.y         // Scroll wheel
```

### Input System (New)

```csharp
// Requires: com.unity.inputsystem package
// Use PlayerInput component or direct:
var keyboard = Keyboard.current;
if (keyboard.spaceKey.wasPressedThisFrame) { }

var mouse = Mouse.current;
Vector2 pos = mouse.position.ReadValue();

var gamepad = Gamepad.current;
Vector2 stick = gamepad?.leftStick.ReadValue() ?? Vector2.zero;
```

---

## Physics

### Physics (3D)

```csharp
// Raycast
if (Physics.Raycast(origin, direction, out RaycastHit hit, maxDistance, layerMask))
{
    hit.point        // World position
    hit.normal       // Surface normal
    hit.collider     // What was hit
    hit.distance     // Distance to hit
}

// Raycast all
RaycastHit[] hits = Physics.RaycastAll(origin, direction, maxDistance);

// Sphere/Box cast (thick raycast)
Physics.SphereCast(origin, radius, direction, out hit, maxDistance);
Physics.BoxCast(center, halfExtents, direction, out hit);

// Overlap (what's in this area?)
Collider[] colliders = Physics.OverlapSphere(center, radius, layerMask);
Collider[] colliders = Physics.OverlapBox(center, halfExtents);

// Settings
Physics.gravity = new Vector3(0, -20, 0);
Physics.IgnoreCollision(colliderA, colliderB, ignore: true);
Physics.IgnoreLayerCollision(layer1, layer2, ignore: true);
```

### Physics2D

```csharp
// Same patterns, 2D versions
RaycastHit2D hit = Physics2D.Raycast(origin, direction, distance, layerMask);
Collider2D[] cols = Physics2D.OverlapCircleAll(point, radius, layerMask);
Collider2D col = Physics2D.OverlapPoint(point, layerMask);
```

---

## Scene & Objects

### SceneManager

```csharp
using UnityEngine.SceneManagement;

// Load
SceneManager.LoadScene("SceneName");
SceneManager.LoadScene(buildIndex);
SceneManager.LoadScene("Scene", LoadSceneMode.Additive);

// Async load
var op = SceneManager.LoadSceneAsync("Scene", LoadSceneMode.Additive);
op.allowSceneActivation = false;  // Control when it activates

// Unload
SceneManager.UnloadSceneAsync("Scene");

// Query
Scene active = SceneManager.GetActiveScene();
int count = SceneManager.sceneCount;
SceneManager.SetActiveScene(scene);
```

### Object (UnityEngine.Object)

```csharp
// Instantiate
GameObject clone = Instantiate(prefab);
GameObject clone = Instantiate(prefab, position, rotation);
GameObject clone = Instantiate(prefab, parent);
Enemy enemy = Instantiate(enemyPrefab);  // Typed

// Destroy
Destroy(gameObject);           // End of frame
Destroy(gameObject, 2f);       // After delay
DestroyImmediate(gameObject);  // Now (Editor only)

// Find (use sparingly — cache results)
GameObject obj = GameObject.Find("Name");
GameObject obj = GameObject.FindWithTag("Player");
GameObject[] objs = GameObject.FindGameObjectsWithTag("Enemy");
Player player = FindObjectOfType<Player>();
Enemy[] enemies = FindObjectsOfType<Enemy>();
```

---

## Resources & Assets

### Resources

```csharp
// Load from Resources/ folder (path without extension)
GameObject prefab = Resources.Load<GameObject>("Prefabs/Enemy");
Sprite sprite = Resources.Load<Sprite>("Sprites/Icon");
TextAsset json = Resources.Load<TextAsset>("Data/config");

// Load all
Sprite[] sprites = Resources.LoadAll<Sprite>("Sprites/Icons");

// Async
ResourceRequest request = Resources.LoadAsync<GameObject>("Prefabs/Enemy");
yield return request;
GameObject prefab = request.asset as GameObject;

// Unload
Resources.UnloadAsset(asset);
Resources.UnloadUnusedAssets();
```

> ⚠️ **Prefer [SerializeField] references over Resources.Load.** Resources folder contents always ship with build.

### Addressables (Package)

```csharp
// Requires: com.unity.addressables package
using UnityEngine.AddressableAssets;

// Load by address
var handle = Addressables.LoadAssetAsync<GameObject>("enemy_prefab");
yield return handle;
GameObject prefab = handle.Result;

// Instantiate directly
Addressables.InstantiateAsync("enemy_prefab", position, rotation);

// Release when done
Addressables.Release(handle);
```

---

## Math & Random

### Mathf

```csharp
// Clamp
Mathf.Clamp(value, min, max)
Mathf.Clamp01(value)              // Clamp 0-1

// Interpolation
Mathf.Lerp(a, b, t)               // Linear interpolate
Mathf.LerpUnclamped(a, b, t)      // t can exceed 0-1
Mathf.InverseLerp(a, b, value)    // Get t from value
Mathf.SmoothStep(a, b, t)         // Smooth ease

// Common
Mathf.Abs(value)
Mathf.Sign(value)                 // -1 or 1
Mathf.Min(a, b) / Mathf.Max(a, b)
Mathf.Floor(f) / Mathf.Ceil(f) / Mathf.Round(f)
Mathf.Pow(base, exp)
Mathf.Sqrt(value)

// Angles
Mathf.Sin(radians) / Mathf.Cos(radians)
Mathf.Deg2Rad / Mathf.Rad2Deg
Mathf.DeltaAngle(from, to)        // Shortest angle difference
Mathf.LerpAngle(a, b, t)          // Lerp handling wrap

// Movement
Mathf.MoveTowards(current, target, maxDelta)
Mathf.SmoothDamp(current, target, ref velocity, smoothTime)

// Constants
Mathf.PI
Mathf.Infinity
Mathf.Epsilon
```

### Random

```csharp
Random.Range(0, 10)          // Int: 0-9 (exclusive max)
Random.Range(0f, 1f)         // Float: 0-1 (inclusive)
Random.value                 // Float 0-1
Random.insideUnitCircle      // Vector2 in circle
Random.insideUnitSphere      // Vector3 in sphere
Random.onUnitSphere          // Vector3 on sphere surface
Random.rotation              // Random Quaternion

// Seed (for reproducibility)
Random.InitState(seed);
```

---

## Vectors & Quaternions

### Vector3 / Vector2

```csharp
// Constants
Vector3.zero / Vector3.one
Vector3.up / Vector3.down / Vector3.forward / Vector3.right

// Operations
Vector3.Distance(a, b)
Vector3.Dot(a, b)                // -1 to 1, same direction = 1
Vector3.Cross(a, b)              // Perpendicular vector
Vector3.Angle(a, b)              // Degrees between
Vector3.SignedAngle(a, b, axis)  // With direction

// Interpolation
Vector3.Lerp(a, b, t)
Vector3.Slerp(a, b, t)           // Spherical (for rotations)
Vector3.MoveTowards(current, target, maxDelta)
Vector3.SmoothDamp(current, target, ref velocity, smoothTime)

// Utility
vector.normalized                // Unit length
vector.magnitude                 // Length
vector.sqrMagnitude              // Faster length comparison
Vector3.Project(vector, onNormal)
Vector3.Reflect(direction, normal)
```

### Quaternion

```csharp
// Identity (no rotation)
Quaternion.identity

// Create
Quaternion.Euler(0, 90, 0)                    // From angles
Quaternion.AngleAxis(45, Vector3.up)          // Around axis
Quaternion.LookRotation(forward, up)          // Face direction

// Interpolation
Quaternion.Lerp(a, b, t)
Quaternion.Slerp(a, b, t)                     // Smoother
Quaternion.RotateTowards(from, to, maxDegrees)

// Operations
rotation * Vector3.forward                     // Rotate a vector
rotationA * rotationB                          // Combine rotations
Quaternion.Inverse(rotation)                   // Reverse
Quaternion.Angle(a, b)                         // Degrees between
```

---

## Coroutines (Yielding)

### WaitFor Classes

```csharp
yield return null;                              // Next frame
yield return new WaitForSeconds(1f);            // Real time
yield return new WaitForSecondsRealtime(1f);    // Ignores timeScale
yield return new WaitForFixedUpdate();          // Next FixedUpdate
yield return new WaitForEndOfFrame();           // After rendering
yield return new WaitUntil(() => condition);    // Until true
yield return new WaitWhile(() => condition);    // While true
yield return StartCoroutine(OtherCoroutine());  // Wait for coroutine
yield return asyncOperation;                    // Wait for async op
```

---

## PlayerPrefs

```csharp
// Simple persistent storage (don't use for sensitive data)
PlayerPrefs.SetInt("HighScore", 100);
PlayerPrefs.SetFloat("Volume", 0.8f);
PlayerPrefs.SetString("PlayerName", "Hero");

int score = PlayerPrefs.GetInt("HighScore", defaultValue: 0);
float vol = PlayerPrefs.GetFloat("Volume", 1f);
string name = PlayerPrefs.GetString("PlayerName", "Player");

PlayerPrefs.HasKey("HighScore")
PlayerPrefs.DeleteKey("HighScore")
PlayerPrefs.DeleteAll()
PlayerPrefs.Save()                // Usually auto-saves
```

> ⚠️ **Use for settings only.** For game saves, use JSON to `persistentDataPath`.

---

## Screen & Display

### Screen

```csharp
Screen.width / Screen.height           // Current resolution
Screen.currentResolution               // Full Resolution struct
Screen.resolutions                     // Available resolutions

Screen.fullScreen = true;
Screen.fullScreenMode = FullScreenMode.FullScreenWindow;
Screen.SetResolution(1920, 1080, true);

Screen.sleepTimeout = SleepTimeout.NeverSleep;  // Mobile
Screen.orientation = ScreenOrientation.Landscape;
```

### Cursor

```csharp
Cursor.visible = false;
Cursor.lockState = CursorLockMode.Locked;   // FPS games
Cursor.lockState = CursorLockMode.Confined; // Keep in window
Cursor.lockState = CursorLockMode.None;     // Normal

Cursor.SetCursor(texture, hotspot, CursorMode.Auto);
```

---

## Gizmos & Handles (Editor)

```csharp
// In OnDrawGizmos() or OnDrawGizmosSelected()
private void OnDrawGizmosSelected()
{
    Gizmos.color = Color.yellow;
    Gizmos.DrawWireSphere(transform.position, radius);
    Gizmos.DrawLine(start, end);
    Gizmos.DrawCube(center, size);
    Gizmos.DrawRay(origin, direction);
    Gizmos.DrawIcon(position, "icon.png");
}
```

---

## Quick Reference Table

| Need              | Use                                 |
| ----------------- | ----------------------------------- |
| Save file path    | `Application.persistentDataPath`    |
| Frame time        | `Time.deltaTime`                    |
| Pause game        | `Time.timeScale = 0`                |
| Log message       | `Debug.Log()`                       |
| Key pressed       | `Input.GetKeyDown()`                |
| Mouse position    | `Input.mousePosition`               |
| Raycast           | `Physics.Raycast()`                 |
| Load scene        | `SceneManager.LoadScene()`          |
| Spawn object      | `Instantiate()`                     |
| Find object       | `FindObjectOfType<T>()`             |
| Random number     | `Random.Range()`                    |
| Clamp value       | `Mathf.Clamp()`                     |
| Smooth movement   | `Vector3.Lerp()` or `SmoothDamp()`  |
| Wait in coroutine | `yield return new WaitForSeconds()` |
| Save settings     | `PlayerPrefs.Set...()`              |
| Hide cursor       | `Cursor.visible = false`            |