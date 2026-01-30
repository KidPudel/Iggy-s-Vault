# Essential Unity Packages & Assets

> Free tools that significantly improve workflow. Built-in or one click install.

---

## Built-in Packages (Package Manager)

Window → Package Manager → Unity Registry

---

### Input System

**What:** Modern input handling with action maps, rebinding, and multi-device support.

**Why:** Legacy Input Manager is limited. This supports gamepads, touch, rebinding UI, and local multiplayer properly.

**Install:** `com.unity.inputsystem`

```csharp
// Create Input Actions asset, then:
public class Player : MonoBehaviour
{
    public void OnMove(InputValue value)
    {
        Vector2 input = value.Get<Vector2>();
    }
    
    public void OnJump(InputValue value)
    {
        if (value.isPressed) Jump();
    }
}
```

---

### TextMeshPro

**What:** High-quality text rendering with rich formatting.

**Why:** Legacy Text component is blurry and limited. TMP is crisp at any size.

**Install:** `com.unity.textmeshpro` (often auto-included)

```csharp
using TMPro;

[SerializeField] private TextMeshProUGUI _text;

_text.text = "Score: <color=yellow>100</color>";
_text.fontSize = 24;
```

---

### Cinemachine

**What:** Smart camera system with follow, framing, zones, and transitions.

**Why:** Writing good camera code is hard. Cinemachine handles follow, look-ahead, screen shake, cutscenes, and transitions.

**Install:** `com.unity.cinemachine`

**Setup:**

1. Add `CinemachineBrain` to Main Camera
2. Create Virtual Cameras (Cinemachine → Create Virtual Camera)
3. Set Follow and Look At targets
4. Cinemachine handles the rest

```
No code needed for basic use. Just configure in Inspector.
```

---

### ProBuilder

**What:** In-editor 3D modeling for level design.

**Why:** Prototype levels without leaving Unity. No Blender round-trips for blockouts.

**Install:** `com.unity.probuilder`

**Use:** Tools → ProBuilder → ProBuilder Window

---

### 2D Packages

|Package|Purpose|
|---|---|
|`com.unity.2d.sprite`|Sprite tools (slicing, atlas)|
|`com.unity.2d.tilemap`|Tile-based level design|
|`com.unity.2d.animation`|Skeletal 2D animation|
|`com.unity.2d.psdimporter`|Import Photoshop files with layers|

---

### Addressables

**What:** Asset management system for loading, unloading, and remote content.

**Why:** Better than Resources folder. Load assets on demand, reduce memory, enable DLC/patches.

**Install:** `com.unity.addressables`

```csharp
// Mark assets as Addressable in Inspector, then:
var handle = Addressables.LoadAssetAsync<GameObject>("EnemyPrefab");
yield return handle;
Instantiate(handle.Result);
```

---

### Localization

**What:** Multi-language support with string tables.

**Why:** Proper localization without hardcoded strings. Handles fonts, plurals, formatting.

**Install:** `com.unity.localization`

---

### Test Framework

**What:** Unit testing and Play Mode testing.

**Why:** Test your systems without manually playing. Catch regressions.

**Install:** `com.unity.test-framework`

```csharp
using NUnit.Framework;

public class HealthTests
{
    [Test]
    public void TakeDamage_ReducesHealth()
    {
        var health = new HealthSystem();
        health.Register(1, 100);
        
        health.Damage(1, 30);
        
        Assert.AreEqual(70, health.Get(1));
    }
}
```

---

### Recorder

**What:** Record gameplay to video/image sequences.

**Why:** Capture trailers, GIFs, bug reproductions without external software.

**Install:** `com.unity.recorder`

**Use:** Window → General → Recorder → Recorder Window

---

### Burst Compiler

**What:** High-performance compiler for jobs/ECS code.

**Why:** 10-100x faster code for CPU-intensive operations.

**Install:** `com.unity.burst`

```csharp
[BurstCompile]
struct MyJob : IJob { ... }
```

---

## Free Asset Store Packages

Asset Store: https://assetstore.unity.com (filter by price: Free)

---

### DOTween (Free version)

**What:** Fast, powerful tweening library.

**Why:** Animate anything with one line. Sequences, easing, callbacks. Industry standard.

**Install:** Asset Store → "DOTween (HOTween v2)"

```csharp
using DG.Tweening;

// Move
transform.DOMove(targetPos, duration);

// Scale
transform.DOScale(Vector3.one * 2, 0.5f);

// UI fade
canvasGroup.DOFade(0, 0.3f);

// Chaining
transform.DOMove(pos, 1f)
    .SetEase(Ease.OutBounce)
    .OnComplete(() => Debug.Log("Done!"));

// Sequences
var seq = DOTween.Sequence();
seq.Append(transform.DOMove(posA, 0.5f));
seq.Append(transform.DOScale(2f, 0.3f));
seq.Join(transform.DORotate(new Vector3(0, 180, 0), 0.3f));
```

---

### Newtonsoft JSON (Unity Package)

**What:** Full-featured JSON library (official Unity package of Json.NET).

**Why:** `JsonUtility` can't do dictionaries, polymorphism, or pretty printing. This can.

**Install:** Package Manager → `com.unity.nuget.newtonsoft-json`

```csharp
using Newtonsoft.Json;

// Serialize
string json = JsonConvert.SerializeObject(data, Formatting.Indented);

// Deserialize
SaveData data = JsonConvert.DeserializeObject<SaveData>(json);

// Handles dictionaries, nested objects, polymorphism
```

---

### UniTask

**What:** Efficient async/await for Unity.

**Why:** No GC allocations. Works with Unity's lifecycle. Better than coroutines for async.

**Install:** https://github.com/Cysharp/UniTask (via Package Manager git URL)

```csharp
using Cysharp.Threading.Tasks;

async UniTaskVoid Start()
{
    await UniTask.Delay(1000);  // 1 second
    
    // Wait for button click
    await button.OnClickAsync();
    
    // Wait for scene load
    await SceneManager.LoadSceneAsync("Level1");
    
    // Parallel
    await UniTask.WhenAll(task1, task2);
}
```

---

### NaughtyAttributes

**What:** Inspector enhancements via attributes.

**Why:** Better Inspector UX with minimal effort. Buttons, conditionals, validation.

**Install:** Asset Store → "NaughtyAttributes"

```csharp
using NaughtyAttributes;

public class Enemy : MonoBehaviour
{
    [Header("Stats")]
    [Range(0, 100)] public int health;
    
    [ShowIf("hasArmor")]
    public int armorValue;
    
    [Dropdown("GetAttackTypes")]
    public string attackType;
    
    [Button("Test Damage")]
    private void TestDamage()
    {
        TakeDamage(10);
    }
    
    [ReadOnly]
    public float timeSinceSpawn;
    
    [Required]
    public GameObject target;
    
    [ValidateInput("IsPositive", "Must be positive")]
    public int damage;
}
```

---

### Odin Inspector (Free Validator)

**What:** Powerful Inspector customization (free version has validation).

**Why:** Even free version adds scene validation, project scanning.

**Install:** Asset Store → "Odin Validator"

(Full Odin is paid but exceptional if you need extensive Inspector customization)

---

### Unity Particle Pack

**What:** Collection of ready-to-use particle effects.

**Why:** Professional VFX without creating from scratch. Fire, smoke, magic, impacts.

**Install:** Asset Store → "Unity Particle Pack"

---

### Kenney Assets

**What:** Massive collection of free art (2D, 3D, UI, audio).

**Why:** Prototype with quality assets. CC0 license — use commercially.

**Install:** https://kenney.nl/assets or Asset Store

---

### Starter Assets (First/Third Person)

**What:** Official Unity character controllers.

**Why:** Production-ready player controllers using Input System. Good starting point.

**Install:** Asset Store → "Starter Assets - First Person" or "Third Person"

---

## Comparison: When to Use What

### Tweening

|Option|Use When|
|---|---|
|`Mathf.Lerp` + Update|Simple, few animations|
|Coroutines|Moderate complexity|
|**DOTween**|Multiple animations, easing, sequences|

### JSON

|Option|Use When|
|---|---|
|`JsonUtility`|Simple data, no dictionaries|
|**Newtonsoft**|Complex data, dictionaries, polymorphism|

### Async

|Option|Use When|
|---|---|
|Coroutines|Simple delays, sequences|
|**UniTask**|Complex async, no GC, cancellation|

### Input

|Option|Use When|
|---|---|
|Legacy Input|Quick prototype|
|**Input System**|Production, rebinding, multiple devices|

### Camera

|Option|Use When|
|---|---|
|Manual code|Very simple follow|
|**Cinemachine**|Any real project|

---

## Quick Install Checklist

**Always install (any project):**

- [ ] TextMeshPro
- [ ] Input System
- [ ] Newtonsoft JSON

**Recommended:**

- [ ] Cinemachine
- [ ] DOTween

**For 2D:**

- [ ] 2D Sprite
- [ ] 2D Tilemap
- [ ] 2D Animation

**For testing:**

- [ ] Test Framework

**For polish:**

- [ ] Recorder (trailers)
- [ ] NaughtyAttributes (better Inspector)

---

## Package Manager Sources

|Source|Access|
|---|---|
|Unity Registry|Built-in packages|
|Asset Store|Free/paid assets (import via Package Manager)|
|Git URL|GitHub packages (Package Manager → + → Add from git URL)|
|OpenUPM|Community packages (https://openupm.com)|

### Adding Git Package

Package Manager → + → Add package from git URL:

```
https://github.com/Cysharp/UniTask.git?path=src/UniTask/Assets/Plugins/UniTask
```