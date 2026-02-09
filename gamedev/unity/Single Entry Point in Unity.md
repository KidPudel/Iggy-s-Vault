# Single Entry Point

> [!quote] The Core Insight Don't let Unity decide when things initialize. You decide.

---

## The Problem

Standard Unity approach: objects live in scenes, each runs `Awake()` and `Start()` whenever Unity feels like it.

**Result:**

- Race conditions (ScriptA tries to access ScriptB before it's ready)
- Hidden dependencies (who initializes first?)
- Scene file conflicts (YAML merge hell in version control)
- No control over "game ready" moment

The [[Execution Order in Unity]] document shows workarounds. This pattern eliminates the problem entirely.

---

## The Paradigm Shift

**Traditional:** Scene contains objects → Unity initializes them → hope for the best

**Single Entry Point:** Empty scene → one script instantiates everything → explicit control

```
Traditional                         Single Entry Point
─────────────────────────────────   ─────────────────────────────────
Scene loads                         Scene loads (nearly empty)
    ↓                                   ↓
Unity runs Awake() on 47 objects    GameInitiator.Start() runs
    ↓                                   ↓
??? order ???                        Initialize services (explicit order)
    ↓                                   ↓
Unity runs Start() on 47 objects    Create objects (explicit order)
    ↓                                   ↓
Bugs when order was wrong           Prepare objects (set state)
                                        ↓
                                    Enable gameplay (you control the frame)
```

---

## The Flow

```
┌─────────────────────────────────────────────────────────────────┐
│  1. SCENE LOAD                                                  │
├─────────────────────────────────────────────────────────────────┤
│  Nearly empty scene. Only GameInitiator exists.                 │
│  Fast load, no race conditions possible.                        │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│  2. INITIALIZATION                                              │
├─────────────────────────────────────────────────────────────────┤
│  Pure C# services, SDKs, backend connections.                   │
│  No GameObjects yet. Nothing visual.                            │
│                                                                 │
│  - Analytics SDK                                                │
│  - Input System                                                 │
│  - Audio system                                                 │
│  - Game systems (see [[Layered Architecture in Unity]])         │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│  3. CREATION                                                    │
├─────────────────────────────────────────────────────────────────┤
│  Instantiate prefabs. Objects exist but are DISABLED.           │
│                                                                 │
│  - Player                                                       │
│  - UI Canvas                                                    │
│  - Enemy pools                                                  │
│  - Level geometry (or load level scene additively)              │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│  4. PREPARATION                                                 │
├─────────────────────────────────────────────────────────────────┤
│  Configure the created objects. Still disabled.                 │
│                                                                 │
│  - Move player to spawn point                                   │
│  - Set UI values (level number, score)                          │
│  - Position enemies                                             │
│  - Apply save data                                              │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│  5. BEGIN                                                       │
├─────────────────────────────────────────────────────────────────┤
│  Everything ready. Release control.                             │
│                                                                 │
│  - Hide loading screen                                          │
│  - Enable player                                                │
│  - Enable enemies                                               │
│  - Start gameplay timers                                        │
│  - YOU control the exact frame gameplay starts                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Implementation

### The Initiator

```csharp
public class GameInitiator : MonoBehaviour
{
    [Header("Prefabs")]
    [SerializeField] private Player _playerPrefab;
    [SerializeField] private GameUI _uiPrefab;
    [SerializeField] private LoadingScreen _loadingScreenPrefab;
    
    [Header("Configuration (design-time, static)")]
    [SerializeField] private GameConfig _config;
    
    // Created instances
    private Player _player;
    private GameUI _ui;
    private LoadingScreen _loadingScreen;
    
    // Runtime state (loaded or created)
    private GameState _state;
    
    private async void Start()
    {
        // Show loading immediately
        _loadingScreen = Instantiate(_loadingScreenPrefab);
        
        // 0. Load runtime state
        _state = SaveSystem.Load() ?? new GameState();
        
        // 1. Initialize (services, pure C#)
        await InitializeServices();
        
        // 2. Create (instantiate, keep disabled)
        await CreateObjects();
        
        // 3. Prepare (configure state)
        PrepareObjects();
        
        // 4. Begin (enable, start gameplay)
        BeginGame();
    }
    
    private async UniTask InitializeServices()
    {
        // Pure C# systems first
        GameSystems.Initialize();
        
        // Third-party SDKs
        await AnalyticsService.Initialize();
        
        // Backend connection
        await BackendService.Connect();
    }
    
    private async UniTask CreateObjects()
    {
        // Load level scene additively
        await SceneManager.LoadSceneAsync($"Level{_state.CurrentLevel}", LoadSceneMode.Additive);
        
        // Instantiate prefabs (disabled by default or disable after)
        _player = Instantiate(_playerPrefab);
        _player.gameObject.SetActive(false);
        
        _ui = Instantiate(_uiPrefab);
        _ui.gameObject.SetActive(false);
    }
    
    private void PrepareObjects()
    {
        // Configure without triggering gameplay logic
        var spawnPoint = FindObjectOfType<SpawnPoint>();
        _player.transform.position = spawnPoint.Position;
        
        // Config = static design-time values
        _player.SetMaxHealth(_config.PlayerMaxHealth);
        
        // State = runtime progress
        _ui.SetLevelText(_state.CurrentLevel);
        _ui.SetScore(_state.Score);
    }
    
    private void BeginGame()
    {
        // Enable in controlled order
        _ui.gameObject.SetActive(true);
        _player.gameObject.SetActive(true);
        
        // Trigger any "round start" logic
        GameEvents.RaiseGameStarted();
        
        // Remove loading screen last
        Destroy(_loadingScreen.gameObject);
    }
}
```

### Config vs State

The Initiator needs both:

```csharp
// CONFIGURATION — ScriptableObject, static, design-time values
// Lives in Assets/Data/GameConfig.asset
[CreateAssetMenu(menuName = "Game/Config")]
public class GameConfig : ScriptableObject
{
    [field: SerializeField] public int PlayerMaxHealth { get; private set; } = 100;
    [field: SerializeField] public float BaseDifficulty { get; private set; } = 1f;
    [field: SerializeField] public int StartingLives { get; private set; } = 3;
}

// STATE — plain C# class, mutable, runtime values
// Loaded from save file or created fresh
[System.Serializable]
public class GameState
{
    public int CurrentLevel = 1;
    public int Score = 0;
    public int Lives = 3;
    public Vector3 LastCheckpoint;
    public List<string> CollectedItems = new();
}
```

**Config** answers: "What are the rules?" (never changes at runtime) **State** answers: "Where is the player in their journey?" (changes constantly)

See [[Data-Driven Design in Unity]] for the full Data/State separation pattern.

### Prefab Bindings

The Initiator holds references to **all** prefabs the game needs:

```csharp
[Header("Core")]
[SerializeField] private Player _playerPrefab;
[SerializeField] private GameUI _uiPrefab;

[Header("Enemies")]
[SerializeField] private Enemy _goblinPrefab;
[SerializeField] private Enemy _orcPrefab;

[Header("Effects")]
[SerializeField] private ParticleSystem _explosionPrefab;
```

**Benefits:**

- Single manifest of all game assets
- Swap prefab variants in one place
- Clear dependencies (if it's not here, it doesn't exist)

---

## Empty Scene Pattern

### Why Empty?

**Performance:** Minimal scene = instant load. Heavy content loads async with progress feedback.

**Version Control:** Empty scene = tiny YAML file. No merge conflicts. Developers edit individual prefabs, not the scene.

**Determinism:** Nothing exists until you create it. No "phantom objects" or forgotten test data.

### Scene Structure

```
Boot.unity (index 0)
├── GameInitiator (the only object)
└── (nothing else)

Level01.unity (loaded additively)
├── Environment geometry
├── Spawn points
└── Level-specific triggers
```

The Initiator lives in Boot. Level content lives in separate scenes loaded additively.

See [[Scene Management in Unity]] for additive loading patterns.

---

## Separation of Creation and Logic

The pattern forces you to split "spawn and configure" methods:

```csharp
// BAD: God method that does everything
public void SpawnEnemiesAroundPoint(Vector3 point, int count)
{
    for (int i = 0; i < count; i++)
    {
        var enemy = Instantiate(_enemyPrefab);               // Creation
        enemy.transform.position = GetRandomPosition(point); // Preparation
        enemy.gameObject.SetActive(true);                    // Begin
        enemy.StartPatrol();                                 // Logic
    }
}

// GOOD: Separate concerns
public class EnemySpawner
{
    private List<Enemy> _pool;
    
    // Called during Creation phase
    public void CreatePool(int size)
    {
        _pool = new List<Enemy>(size);
        for (int i = 0; i < size; i++)
        {
            var enemy = Instantiate(_enemyPrefab);
            enemy.gameObject.SetActive(false);
            _pool.Add(enemy);
        }
    }
    
    // Called during Preparation phase
    public void PositionAroundPoint(Vector3 point)
    {
        foreach (var enemy in _pool)
            enemy.transform.position = GetRandomPosition(point);
    }
    
    // Called during Begin phase
    public void ActivateAll()
    {
        foreach (var enemy in _pool)
        {
            enemy.gameObject.SetActive(true);
            enemy.StartPatrol();
        }
    }
}
```

This is Separation of Concerns enforced by architecture.

---

## Async Flow

Because the entry point is `async`, you can await anything:

```csharp
private async void Start()
{
    _loadingScreen.Show();
    
    // Wait for server
    await BackendService.Connect();
    _loadingScreen.SetProgress(0.2f);
    
    // Wait for asset bundles
    await AssetBundleLoader.LoadAllAsync();
    _loadingScreen.SetProgress(0.5f);
    
    // Wait for scene
    await SceneManager.LoadSceneAsync("Level01", LoadSceneMode.Additive);
    _loadingScreen.SetProgress(0.8f);
    
    // Synchronous setup
    CreateObjects();
    PrepareObjects();
    _loadingScreen.SetProgress(1f);
    
    // Dramatic pause for loading screen animation
    await UniTask.Delay(500);
    
    BeginGame();
}
```

In traditional `Start/Awake` chaos, coordinating these delays across objects is nearly impossible.

---

## Dependency Injection (Lite)

The pattern naturally inverts dependencies:

```csharp
// OLD: LevelUI searches for its dependencies
public class LevelUI : MonoBehaviour
{
    private void Start()
    {
        var levelManager = FindObjectOfType<LevelManager>();
        _levelText.text = $"Level {levelManager.CurrentLevel}";
    }
}

// NEW: Initiator injects what LevelUI needs
public class LevelUI : MonoBehaviour
{
    public void SetLevel(int level)
    {
        _levelText.text = $"Level {level}";
    }
}

// In Initiator (using runtime state, not config):
_ui.SetLevel(_state.CurrentLevel);
```

LevelUI no longer knows LevelManager or GameState exist. It just receives data.

---

## Integration with Layered Architecture

Single Entry Point and [[Layered Architecture in Unity]] complement each other:

```csharp
private async UniTask InitializeServices()
{
    // Create Systems (pure C# — see Layered Architecture)
    var healthSystem = new HealthSystem();
    var inventorySystem = new InventorySystem();
    var questSystem = new QuestSystem();
    
    // Register for global access
    Services.Register(healthSystem);
    Services.Register(inventorySystem);
    Services.Register(questSystem);
    
    // Systems exist before any GameObjects
}

private async UniTask CreateObjects()
{
    // Components can now safely access Systems in their Awake/OnEnable
    _player = Instantiate(_playerPrefab);
    // Player.OnEnable() can call Services.Get<HealthSystem>() — system exists
}
```

**Layered Architecture** defines runtime responsibilities. **Single Entry Point** ensures everything exists in the right order.

---

## Integration with Dependency Injection

When using a DI framework (VContainer), the container replaces manual Service Locator registration:

```csharp
// The LifetimeScope IS your Single Entry Point for registration
public class GameLifetimeScope : LifetimeScope
{
    [SerializeField] private GameConfig _config;
    
    protected override void Configure(IContainerBuilder builder)
    {
        // Plain C# systems — container CREATES and OWNS these
        builder.Register<HealthSystem>(Lifetime.Singleton);
        builder.Register<InventorySystem>(Lifetime.Singleton);
        builder.Register<QuestSystem>(Lifetime.Singleton);
        
        // MonoBehaviours in Persistent scene — container FINDS these
        // (You created them in the scene; container wires them up)
        builder.RegisterComponentInHierarchy<AudioManager>().As<IAudioService>();
        
        // Config — just make it available for injection
        builder.RegisterInstance(_config);
        
        // Entry points — container calls these after building
        builder.RegisterEntryPoint<GameInitiator>();
    }
}

// GameInitiator receives dependencies via injection
public class GameInitiator : IAsyncStartable
{
    private readonly HealthSystem _health;
    private readonly IAudioService _audio;
    private readonly GameConfig _config;
    
    public GameInitiator(HealthSystem health, IAudioService audio, GameConfig config)
    {
        _health = health;
        _audio = audio;
        _config = config;
    }
    
    public async UniTask StartAsync(CancellationToken cancellation)
    {
        // All dependencies already injected — no manual registration needed
        await LoadLevelAsync();
        // ...
    }
}
```

**Key insight:** The DI container handles _what_ gets created and _who_ gets what. The Single Entry Point pattern (phases, explicit ordering) still applies — it's just automated.

|Without DI|With DI|
|---|---|
|Manual `new HealthSystem()`|`builder.Register<HealthSystem>()`|
|Manual `Services.Register()`|Container tracks automatically|
|Manual `Services.Get<T>()` in Awake|`[Inject]` or constructor injection|
|You control creation order|Container resolves dependency graph|

See [[Reference Objects in Unity]] for DI lifetime details and problem of Singleton.

---

## When to Use

**Use Single Entry Point when:**

- Team has merge conflicts in scene files
- Initialization race conditions cause bugs
- You need loading screens with progress
- Game requires server connection before gameplay
- You want deterministic startup

**Skip it for:**

- Small prototypes
- Game jams
- Solo projects where scene conflicts don't matter
- Simple games without loading requirements

---

## Scaling Concerns

For large projects, the Initiator can become a God Class. Solutions:

**Sub-Initiators:**

```csharp
private async UniTask CreateObjects()
{
    await _enemyInitiator.Initialize();
    await _uiInitiator.Initialize();
    await _audioInitiator.Initialize();
}
```

**State Machine:**

```csharp
public class GameInitiator : MonoBehaviour
{
    private IInitState _currentState;
    
    private void Start()
    {
        _currentState = new LoadingState(this);
        _currentState.Enter();
    }
}
```

**Addressables/Asset Management:** For very large games, combine with Addressables to load content on demand rather than all upfront.

---

## Checklist

**Scene Setup:**

- [ ] Boot scene contains only GameInitiator
- [ ] Level content in separate scenes (loaded additively)
- [ ] Prefabs referenced via [SerializeField], not in scene

**Initiator:**

- [ ] Single `Start()` method controls all initialization
- [ ] Async flow for anything that can fail or delay
- [ ] Clear phases: Initialize → Create → Prepare → Begin

**Objects:**

- [ ] Created disabled (or disabled immediately)
- [ ] Configured before enabling
- [ ] No `Start()` logic that assumes world state

**Sanity:**

- [ ] Can you describe the exact frame gameplay begins?
- [ ] Would a new team member understand the startup flow?
- [ ] Are scene files small and merge-friendly?

---

## Key Takeaways

1. **Empty scene** — only the Initiator exists at load
2. **One entry point** — single `Start()` controls everything
3. **Explicit phases** — Initialize → Create → Prepare → Begin
4. **Objects start disabled** — no premature logic
5. **Async-friendly** — await servers, bundles, scenes
6. **Forces good patterns** — separation of creation and logic
7. **Complements other architecture** — works with Layered Architecture, doesn't replace it

> [!quote] The Test Can you point to the exact line of code where gameplay starts?