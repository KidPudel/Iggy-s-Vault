# Common Patterns Cookbook

> Practical implementations of common game patterns in Unity.

---

## What is This Document?

This is a **reference cookbook** — not a tutorial. Each pattern shows:

1. What problem it solves
2. When to use it (and when not to)
3. A working implementation you can copy and adapt

These patterns appear in almost every Unity project. Knowing them saves you from reinventing wheels.

**How to use this document:** Don't read it start to finish. Skim the pattern names, understand what problems they solve, and come back when you need one.

---

## Singleton

### The Problem

You need exactly *one instance (meaning with own state)* of something (GameManager, AudioManager) accessible from anywhere. Without this pattern, you'd have to pass references through every object or use `FindObjectOfType` repeatedly (slow).

### When to Use

- Managers that must exist once: GameManager, AudioManager, InputManager
- When global access is genuinely needed
- Small projects where testability isn't a priority

### When NOT to Use

- Just because you're too lazy to pass references properly
- In large teams where implicit dependencies cause confusion
- When you need multiple instances or testability

### Implementation

```csharp
public class GameManager : MonoBehaviour
{
    public static GameManager Instance { get; private set; }
    
    private void Awake()
    {
        if (Instance != null)
        {
            Destroy(gameObject);  // Duplicate — destroy myself
            return;
        }
        Instance = this;
    }
    
    private void OnDestroy()
    {
        if (Instance == this)
            Instance = null;  // Clean up static reference
    }
}

// Access from anywhere
GameManager.Instance.StartGame();
```

### Why the Duplicate Check?

If you load a scene containing a GameManager while one already exists, you'd have two. The check ensures only the first survives.

### Better Alternative: Persistent Scene

Instead of singletons, put managers in a Persistent scene that never unloads (see [[Scene Management in Unity]]). You can see managers in the hierarchy, easier to debug.

---

## Object Pool

### The Problem

`Instantiate` and `Destroy` are expensive operations. They:

- Allocate memory (creates garbage)
- Trigger garbage collection (causes stutters)
- Spike the CPU

If you're spawning bullets every 0.1 seconds and destroying them on impact, your game will stutter.

### The Solution

**Reuse objects instead of destroying them.** When a bullet hits something, don't destroy it — deactivate it and put it back in a pool. Next time you need a bullet, grab one from the pool instead of instantiating.

### When to Use

- Bullets, projectiles
- Particle effects
- Frequently spawned enemies
- Anything created/destroyed rapidly (more than a few per second)

### When NOT to Use

- Objects that exist for the entire game
- Objects spawned rarely (a few per minute is fine)
- Prototyping (add pooling later when you need performance)

### Implementation

```csharp
public class ObjectPool<T> where T : Component
{
    private readonly T _prefab;
    private readonly Transform _parent;
    private readonly Queue<T> _pool = new();
    
    public ObjectPool(T prefab, int initialSize, Transform parent = null)
    {
        _prefab = prefab;
        _parent = parent;
        
        // Pre-warm: create objects upfront to avoid stutters later
        for (int i = 0; i < initialSize; i++)
            _pool.Enqueue(CreateNew());
    }
    
    private T CreateNew()
    {
        var obj = Object.Instantiate(_prefab, _parent);
        obj.gameObject.SetActive(false);
        return obj;
    }
    
    public T Get()
    {
        var obj = _pool.Count > 0 ? _pool.Dequeue() : CreateNew();
        obj.gameObject.SetActive(true);
        return obj;
    }
    
    public void Return(T obj)
    {
        obj.gameObject.SetActive(false);
        _pool.Enqueue(obj);
    }
}
```

### Usage

```csharp
public class Gun : MonoBehaviour
{
    [SerializeField] private Bullet _bulletPrefab;
    private ObjectPool<Bullet> _bulletPool;
    
    private void Awake()
    {
        _bulletPool = new ObjectPool<Bullet>(_bulletPrefab, initialSize: 20);
    }
    
    public void Fire()
    {
        var bullet = _bulletPool.Get();
        bullet.transform.position = _muzzle.position;
        bullet.transform.rotation = _muzzle.rotation;
        bullet.Init(OnBulletDone);
    }
    
    private void OnBulletDone(Bullet bullet) => _bulletPool.Return(bullet);
}
```

### The Pooled Object Must Reset Itself

When an object is reused, it must reset to a clean state:

```csharp
public class Bullet : MonoBehaviour
{
    private Action<Bullet> _onDone;
    private float _lifetime;
    
    public void Init(Action<Bullet> onDone)
    {
        _onDone = onDone;
        _lifetime = 0;  // Reset state
    }
    
    private void Update()
    {
        _lifetime += Time.deltaTime;
        if (_lifetime > 5f)
            _onDone?.Invoke(this);  // Return to pool instead of Destroy
    }
    
    private void OnTriggerEnter(Collider other)
    {
        // Deal damage, play effect, etc.
        _onDone?.Invoke(this);  // Return to pool
    }
}
```

---

## State Machine

### The Problem

Complex entities (enemies, players, UI) behave differently based on their current state. Without structure, you get unmaintainable spaghetti:

```csharp
// This becomes impossible to manage
void Update()
{
    if (_isIdle && CanSeePlayer()) { _isIdle = false; _isChasing = true; }
    if (_isChasing && !CanSeePlayer()) { _isChasing = false; _isSearching = true; }
    if (_isChasing && InAttackRange()) { _isChasing = false; _isAttacking = true; }
    if (_isAttacking && !InAttackRange()) { _isAttacking = false; _isChasing = true; }
    // ... every new state multiplies the complexity
}
```

### The Solution

A **state machine** organizes behavior into discrete states. Each state:

- Knows how to behave while active
- Knows when to transition to other states
- Is isolated — you can understand and modify it without touching other states

### When to Use

- Enemy AI (Idle, Patrol, Chase, Attack, Flee)
- Player states (Grounded, Jumping, Falling, Climbing, Swimming)
- UI flows (MainMenu, Playing, Paused, GameOver)
- Anything with distinct modes of behavior

### Implementation

```csharp
public interface IState
{
    void Enter();   // Called when entering this state
    void Update();  // Called every frame while active
    void Exit();    // Called when leaving this state
}

public class StateMachine
{
    public IState CurrentState { get; private set; }
    
    public void ChangeState(IState newState)
    {
        CurrentState?.Exit();
        CurrentState = newState;
        CurrentState?.Enter();
    }
    
    public void Update() => CurrentState?.Update();
}
```

### Example: Enemy AI

```csharp
public class Enemy : MonoBehaviour
{
    public StateMachine StateMachine { get; private set; }
    public Transform Player { get; private set; }
    public Animator Animator { get; private set; }
    
    private void Awake()
    {
        StateMachine = new StateMachine();
        Animator = GetComponent<Animator>();
    }
    
    private void Start()
    {
        Player = FindObjectOfType<Player>().transform;
        StateMachine.ChangeState(new IdleState(this));
    }
    
    private void Update() => StateMachine.Update();
    
    public bool CanSeePlayer() => Vector3.Distance(transform.position, Player.position) < 10f;
    public bool InAttackRange() => Vector3.Distance(transform.position, Player.position) < 2f;
}

public class IdleState : IState
{
    private readonly Enemy _enemy;
    public IdleState(Enemy enemy) => _enemy = enemy;
    
    public void Enter() => _enemy.Animator.Play("Idle");
    public void Exit() { }
    
    public void Update()
    {
        if (_enemy.CanSeePlayer())
            _enemy.StateMachine.ChangeState(new ChaseState(_enemy));
    }
}

public class ChaseState : IState
{
    private readonly Enemy _enemy;
    public ChaseState(Enemy enemy) => _enemy = enemy;
    
    public void Enter() => _enemy.Animator.Play("Run");
    public void Exit() { }
    
    public void Update()
    {
        // Move toward player
        var dir = (_enemy.Player.position - _enemy.transform.position).normalized;
        _enemy.transform.position += dir * 5f * Time.deltaTime;
        
        // Transitions
        if (_enemy.InAttackRange())
            _enemy.StateMachine.ChangeState(new AttackState(_enemy));
        else if (!_enemy.CanSeePlayer())
            _enemy.StateMachine.ChangeState(new IdleState(_enemy));
    }
}
```

### Why This Is Better

- Each state is self-contained — easy to understand
- Adding states doesn't touch existing code
- Easy to debug — just check `CurrentState`
- Transitions are explicit and visible

---

## Health Component

### The Problem

Many things have health: players, enemies, destructibles, bosses. Reimplementing health logic in each is wasteful and error-prone.

### The Solution

A reusable Health component that works on anything. It handles the data (current/max health) and announces changes via events. What happens on death is up to the object using it.

### Implementation

```csharp
public class Health : MonoBehaviour
{
    [SerializeField] private int _max = 100;
    
    public int Current { get; private set; }
    public int Max => _max;
    public bool IsDead => Current <= 0;
    public float Percentage => (float)Current / _max;
    
    public event Action<int, int> OnChanged;  // (current, max)
    public event Action OnDied;
    
    private void Awake() => Current = _max;
    
    public void TakeDamage(int amount)
    {
        if (IsDead) return;
        
        Current = Mathf.Max(0, Current - amount);
        OnChanged?.Invoke(Current, _max);
        
        if (Current <= 0)
            OnDied?.Invoke();
    }
    
    public void Heal(int amount)
    {
        if (IsDead) return;
        
        Current = Mathf.Min(_max, Current + amount);
        OnChanged?.Invoke(Current, _max);
    }
    
    public void SetMax(int newMax, bool heal = false)
    {
        _max = newMax;
        if (heal) Current = _max;
        OnChanged?.Invoke(Current, _max);
    }
}
```

### Why Events?

Health doesn't know or care what happens when damage is taken or health reaches zero. It just announces it. Other components decide how to respond:

```csharp
// Health bar updates when health changes
public class HealthBar : MonoBehaviour
{
    [SerializeField] private Health _health;
    [SerializeField] private Image _fill;
    
    private void OnEnable() => _health.OnChanged += UpdateBar;
    private void OnDisable() => _health.OnChanged -= UpdateBar;
    
    private void UpdateBar(int current, int max) => _fill.fillAmount = (float)current / max;
}

// Enemy dies when health depleted
public class Enemy : MonoBehaviour
{
    private void OnEnable() => GetComponent<Health>().OnDied += Die;
    private void OnDisable() => GetComponent<Health>().OnDied -= Die;
    
    private void Die()
    {
        DropLoot();
        Destroy(gameObject);
    }
}

// Barrel explodes when health depleted
public class ExplosiveBarrel : MonoBehaviour
{
    private void OnEnable() => GetComponent<Health>().OnDied += Explode;
    private void OnDisable() => GetComponent<Health>().OnDied -= Explode;
    
    private void Explode()
    {
        DealAreaDamage();
        SpawnExplosionEffect();
        Destroy(gameObject);
    }
}
```

Same Health component, completely different responses.

---

## Interaction System

### The Problem

Player walks up to objects and presses E to interact: doors, NPCs, items, levers, chests. You need to:

1. Detect what's in range
2. Show appropriate prompt ("Open Door", "Talk", "Pick Up Sword")
3. Trigger the interaction

### The Solution

An **interface** defines what interactable objects must do. An **Interactor** component on the player detects them. Each interactable object implements its own behavior.

### Implementation

```csharp
// Any interactable object implements this
public interface IInteractable
{
    string GetPrompt();                    // "Open", "Talk to Bob", "Pick up Sword"
    void Interact(GameObject interactor);  // Do the thing
}

// Goes on the player
public class Interactor : MonoBehaviour
{
    [SerializeField] private float _range = 2f;
    [SerializeField] private LayerMask _mask;
    
    private IInteractable _current;
    public event Action<IInteractable> OnTargetChanged;
    
    private void Update()
    {
        // Raycast forward to find interactable
        IInteractable found = null;
        if (Physics.Raycast(transform.position, transform.forward, out var hit, _range, _mask))
            found = hit.collider.GetComponent<IInteractable>();
        
        if (found != _current)
        {
            _current = found;
            OnTargetChanged?.Invoke(_current);  // UI listens to show/hide prompt
        }
        
        if (_current != null && Input.GetKeyDown(KeyCode.E))
            _current.Interact(gameObject);
    }
}
```

### Example Interactables

```csharp
public class Door : MonoBehaviour, IInteractable
{
    private bool _isOpen;
    
    public string GetPrompt() => _isOpen ? "Close" : "Open";
    
    public void Interact(GameObject interactor)
    {
        _isOpen = !_isOpen;
        GetComponent<Animator>().Play(_isOpen ? "Open" : "Close");
    }
}

public class ItemPickup : MonoBehaviour, IInteractable
{
    [SerializeField] private ItemData _item;
    
    public string GetPrompt() => $"Pick up {_item.displayName}";
    
    public void Interact(GameObject interactor)
    {
        if (interactor.GetComponent<Inventory>().TryAdd(_item))
            Destroy(gameObject);
    }
}

public class NPC : MonoBehaviour, IInteractable
{
    [SerializeField] private string _name;
    [SerializeField] private DialogueData _dialogue;
    
    public string GetPrompt() => $"Talk to {_name}";
    
    public void Interact(GameObject interactor)
    {
        DialogueManager.Instance.StartDialogue(_dialogue);
    }
}
```

### Why Interface?

The Interactor doesn't know about doors, items, or NPCs. It only knows `IInteractable`. Any new interactable type implements the interface — no changes to Interactor needed. This is the Open/Closed Principle in action.

---

## Save System

### The Problem

Games need to persist progress across sessions: player position, health, inventory, completed quests, unlocked abilities. This data must survive quitting and restarting.

### Key Concepts

1. **SaveData** — A plain C# class containing everything to save
2. **Serialization** — Converting SaveData to JSON (text) so it can be written to a file
3. **Storage** — Writing the file to `Application.persistentDataPath`
4. **Loading** — Reading the file and deserializing back to SaveData

### Why `Application.persistentDataPath`?

This is the correct location for save files on all platforms:

- **Windows:** `C:\Users\<user>\AppData\LocalLow\<company>\<game>\`
- **Mac:** `~/Library/Application Support/<company>/<game>/`
- **Mobile:** App's private storage

It's writable, survives game updates, and works on all platforms.

### Implementation

```csharp
[System.Serializable]
public class SaveData
{
    public Vector3 playerPosition;
    public int playerHealth;
    public List<string> inventoryItemIds;
    public List<string> completedQuestIds;
    public float playTime;
}

public static class SaveSystem
{
    private static string GetPath(string slot) => 
        Path.Combine(Application.persistentDataPath, $"save_{slot}.json");
    
    public static void Save(SaveData data, string slot)
    {
        string json = JsonUtility.ToJson(data, prettyPrint: true);
        File.WriteAllText(GetPath(slot), json);
    }
    
    public static SaveData Load(string slot)
    {
        string path = GetPath(slot);
        if (!File.Exists(path)) return null;
        
        string json = File.ReadAllText(path);
        return JsonUtility.FromJson<SaveData>(json);
    }
    
    public static bool Exists(string slot) => File.Exists(GetPath(slot));
    public static void Delete(string slot) => File.Delete(GetPath(slot));
}
```

### Usage

```csharp
public void SaveGame()
{
    var data = new SaveData
    {
        playerPosition = _player.transform.position,
        playerHealth = _player.Health.Current,
        inventoryItemIds = _inventory.GetItemIds(),
        completedQuestIds = _questManager.GetCompletedIds(),
        playTime = _playTime
    };
    SaveSystem.Save(data, "slot1");
}

public void LoadGame()
{
    var data = SaveSystem.Load("slot1");
    if (data == null) return;
    
    _player.transform.position = data.playerPosition;
    _player.Health.SetCurrent(data.playerHealth);
    _inventory.LoadItems(data.inventoryItemIds);
    _questManager.LoadCompleted(data.completedQuestIds);
    _playTime = data.playTime;
}
```

### JsonUtility Limitations

Unity's `JsonUtility` is fast but limited — no dictionaries, no polymorphism. For complex saves, use **Newtonsoft.Json** (see [[Essential Unity Packages and Assets]]).

---

## Timer

### The Problem

You need cooldowns, delays, and durations everywhere. Scattering `_timer -= Time.deltaTime` throughout your code is messy and error-prone.

### The Solution

Encapsulate timing logic in a reusable Timer class.

### Implementation

```csharp
public class Timer
{
    public float Duration { get; private set; }
    public float Remaining { get; private set; }
    public bool IsComplete => Remaining <= 0;
    public float Progress => 1f - (Remaining / Duration);
    
    public event Action OnComplete;
    
    public Timer(float duration)
    {
        Duration = duration;
        Remaining = duration;
    }
    
    public void Tick(float deltaTime)
    {
        if (IsComplete) return;
        
        Remaining -= deltaTime;
        if (Remaining <= 0)
        {
            Remaining = 0;
            OnComplete?.Invoke();
        }
    }
    
    public void Reset() => Remaining = Duration;
    public void Reset(float newDuration)
    {
        Duration = newDuration;
        Remaining = newDuration;
    }
}
```

### Usage

```csharp
public class Weapon : MonoBehaviour
{
    [SerializeField] private float _cooldownDuration = 0.5f;
    private Timer _cooldown;
    
    private void Awake() => _cooldown = new Timer(_cooldownDuration);
    
    private void Update()
    {
        _cooldown.Tick(Time.deltaTime);
        
        if (Input.GetButton("Fire") && _cooldown.IsComplete)
        {
            Fire();
            _cooldown.Reset();
        }
    }
}
```

### Why Not Just Use Invoke?

`Invoke("MethodName", delay)` exists but:

- Uses strings (no compile-time checking)
- Can't check remaining time or progress
- Can't pause or reset
- Hard to cancel

Timer is explicit and controllable.

---

## Camera Follow

### The Problem

Camera needs to follow the player smoothly without jittering.

### The Key Insight: Use LateUpdate

Camera follow must happen in `LateUpdate`. Why? The player moves in `Update`. If the camera also moves in `Update`, sometimes the camera moves first (before player), sometimes after — causing jitter. `LateUpdate` guarantees the camera moves after all `Update` calls finish.

### Implementation

```csharp
public class CameraFollow : MonoBehaviour
{
    [SerializeField] private Transform _target;
    [SerializeField] private Vector3 _offset = new Vector3(0, 5, -10);
    [SerializeField] private float _smoothSpeed = 5f;
    
    private void LateUpdate()
    {
        if (_target == null) return;
        
        Vector3 desired = _target.position + _offset;
        transform.position = Vector3.Lerp(transform.position, desired, _smoothSpeed * Time.deltaTime);
        transform.LookAt(_target);
    }
}
```

### For Real Projects: Use Cinemachine

This script works for prototypes. For production, use **Cinemachine** — it handles follow, collision avoidance, screen shake, and cinematic transitions. See [[Essential Unity Packages and Assets]].

---

## Simple Tweening

### The Problem

You want to smoothly animate values over time: move to position, fade out, scale up. Writing coroutines for each case is tedious.

### Implementation

```csharp
public static class Tween
{
    public static IEnumerator MoveTo(Transform t, Vector3 target, float duration)
    {
        Vector3 start = t.position;
        float elapsed = 0;
        
        while (elapsed < duration)
        {
            elapsed += Time.deltaTime;
            t.position = Vector3.Lerp(start, target, elapsed / duration);
            yield return null;
        }
        t.position = target;
    }
    
    public static IEnumerator FadeCanvasGroup(CanvasGroup group, float target, float duration)
    {
        float start = group.alpha;
        float elapsed = 0;
        
        while (elapsed < duration)
        {
            elapsed += Time.deltaTime;
            group.alpha = Mathf.Lerp(start, target, elapsed / duration);
            yield return null;
        }
        group.alpha = target;
    }
    
    public static IEnumerator ScaleTo(Transform t, Vector3 target, float duration)
    {
        Vector3 start = t.localScale;
        float elapsed = 0;
        
        while (elapsed < duration)
        {
            elapsed += Time.deltaTime;
            t.localScale = Vector3.Lerp(start, target, elapsed / duration);
            yield return null;
        }
        t.localScale = target;
    }
}

// Usage
StartCoroutine(Tween.MoveTo(transform, targetPos, 0.5f));
StartCoroutine(Tween.FadeCanvasGroup(canvasGroup, 0f, 0.3f));
```

### For Real Projects: Use DOTween

DOTween is the industry standard. Much cleaner syntax:

```csharp
transform.DOMove(targetPos, 0.5f).SetEase(Ease.OutQuad);
canvasGroup.DOFade(0f, 0.3f).OnComplete(() => Destroy(gameObject));

DOTween.Sequence()
    .Append(transform.DOMove(pos, 0.5f))
    .Append(transform.DOScale(2f, 0.3f));
```

See [[Essential Unity Packages and Assets]].

---

## Spawner

### The Problem

You need to create enemies, pickups, or obstacles at runtime at specific positions.

### Implementation

```csharp
public class Spawner : MonoBehaviour
{
    [SerializeField] private GameObject _prefab;
    [SerializeField] private Transform[] _spawnPoints;
    
    public GameObject SpawnAtRandom()
    {
        var point = _spawnPoints[Random.Range(0, _spawnPoints.Length)];
        return Instantiate(_prefab, point.position, point.rotation);
    }
    
    public GameObject SpawnAt(int index)
    {
        var point = _spawnPoints[index];
        return Instantiate(_prefab, point.position, point.rotation);
    }
}
```

### Wave Spawner

```csharp
public class WaveSpawner : MonoBehaviour
{
    [SerializeField] private GameObject _enemyPrefab;
    [SerializeField] private Transform[] _spawnPoints;
    [SerializeField] private int _enemiesPerWave = 5;
    [SerializeField] private float _timeBetweenSpawns = 0.5f;
    
    public IEnumerator SpawnWave()
    {
        for (int i = 0; i < _enemiesPerWave; i++)
        {
            var point = _spawnPoints[Random.Range(0, _spawnPoints.Length)];
            Instantiate(_enemyPrefab, point.position, point.rotation);
            yield return new WaitForSeconds(_timeBetweenSpawns);
        }
    }
}
```

For high-frequency spawning, combine with Object Pool.

---

## Audio Manager

### The Problem

You want to play sounds from anywhere without every object needing its own AudioSource setup.

### Implementation

```csharp
public class AudioManager : MonoBehaviour
{
    public static AudioManager Instance { get; private set; }
    
    [SerializeField] private AudioSource _musicSource;
    [SerializeField] private AudioSource _sfxSource;
    
    private void Awake() => Instance = this;
    
    public void PlaySFX(AudioClip clip, float volume = 1f)
    {
        _sfxSource.PlayOneShot(clip, volume);
    }
    
    public void PlaySFXAt(AudioClip clip, Vector3 position, float volume = 1f)
    {
        AudioSource.PlayClipAtPoint(clip, position, volume);
    }
    
    public void PlayMusic(AudioClip clip, bool loop = true)
    {
        _musicSource.clip = clip;
        _musicSource.loop = loop;
        _musicSource.Play();
    }
    
    public void StopMusic() => _musicSource.Stop();
}

// Usage
AudioManager.Instance.PlaySFX(shootSound);
AudioManager.Instance.PlaySFXAt(explosionSound, transform.position);
```

### Why PlayOneShot?

`AudioSource.Play()` stops the current sound and plays the new one. `PlayOneShot()` allows overlapping — multiple gunshots, footsteps, etc.

---

## Quick Reference: When to Use What

|Pattern|Use When|
|---|---|
|Singleton|Need global access to a single instance|
|Object Pool|Spawning/destroying objects rapidly|
|State Machine|Entity has distinct behavior modes|
|Health Component|Anything that can be damaged|
|Interaction System|Player interacts with world objects|
|Save System|Need to persist progress|
|Timer|Cooldowns, delays, durations|
|Camera Follow|Camera tracks a target|
|Tweening|Smooth value animation over time|
|Spawner|Creating objects at specific points|
|Audio Manager|Playing sounds from anywhere|**