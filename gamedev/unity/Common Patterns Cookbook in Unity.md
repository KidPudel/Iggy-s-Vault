# Common Patterns Cookbook

> Practical implementations of common game patterns in Unity.

---

## Singleton

```csharp
public class GameManager : MonoBehaviour
{
    public static GameManager Instance { get; private set; }
    
    private void Awake()
    {
        if (Instance != null)
        {
            Destroy(gameObject);
            return;
        }
        Instance = this;
    }
}

// Usage
GameManager.Instance.DoSomething();
```

For persistent singletons, use a dedicated scene instead of `DontDestroyOnLoad`. See [[Scene Management in Unity]].

---

## Object Pool

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
        
        for (int i = 0; i < initialSize; i++)
            AddToPool(CreateNew());
    }
    
    private T CreateNew()
    {
        var obj = Object.Instantiate(_prefab, _parent);
        obj.gameObject.SetActive(false);
        return obj;
    }
    
    private void AddToPool(T obj)
    {
        obj.gameObject.SetActive(false);
        _pool.Enqueue(obj);
    }
    
    public T Get()
    {
        var obj = _pool.Count > 0 ? _pool.Dequeue() : CreateNew();
        obj.gameObject.SetActive(true);
        return obj;
    }
    
    public void Return(T obj) => AddToPool(obj);
}

// Usage
private ObjectPool<Bullet> _bulletPool;

private void Awake()
{
    _bulletPool = new ObjectPool<Bullet>(_bulletPrefab, 20, transform);
}

public void Fire()
{
    var bullet = _bulletPool.Get();
    bullet.transform.position = _muzzle.position;
    bullet.Init(() => _bulletPool.Return(bullet));
}
```

---

## Spawning

```csharp
public class Spawner : MonoBehaviour
{
    [SerializeField] private Enemy _enemyPrefab;
    [SerializeField] private Transform[] _spawnPoints;
    
    public Enemy Spawn()
    {
        var point = _spawnPoints[Random.Range(0, _spawnPoints.Length)];
        var enemy = Instantiate(_enemyPrefab, point.position, point.rotation);
        return enemy;
    }
    
    public Enemy Spawn(Vector3 position)
    {
        return Instantiate(_enemyPrefab, position, Quaternion.identity);
    }
}
```

---

## Simple Inventory

```csharp
// Item data (ScriptableObject)
[CreateAssetMenu(menuName = "Game/ItemData")]
public class ItemData : ScriptableObject
{
    public string id;
    public string displayName;
    public Sprite icon;
    public int maxStack;
}

// Item instance (runtime state)
[System.Serializable]
public class ItemStack
{
    public string itemId;
    public int quantity;
    
    [System.NonSerialized] private ItemData _data;
    public ItemData Data => _data ??= ItemDatabase.Get(itemId);
    
    public ItemStack(ItemData data, int qty)
    {
        itemId = data.id;
        _data = data;
        quantity = qty;
    }
}

// Inventory
public class Inventory : MonoBehaviour
{
    [SerializeField] private int _capacity = 20;
    
    private List<ItemStack> _items = new();
    
    public IReadOnlyList<ItemStack> Items => _items;
    public event Action OnChanged;
    
    public bool TryAdd(ItemData data, int quantity = 1)
    {
        // Try stack existing
        var existing = _items.FirstOrDefault(i => i.itemId == data.id && i.quantity < data.maxStack);
        if (existing != null)
        {
            int canAdd = Mathf.Min(quantity, data.maxStack - existing.quantity);
            existing.quantity += canAdd;
            quantity -= canAdd;
        }
        
        // Add new stacks
        while (quantity > 0 && _items.Count < _capacity)
        {
            int stackSize = Mathf.Min(quantity, data.maxStack);
            _items.Add(new ItemStack(data, stackSize));
            quantity -= stackSize;
        }
        
        OnChanged?.Invoke();
        return quantity == 0;
    }
    
    public bool Remove(string itemId, int quantity = 1)
    {
        var stack = _items.FirstOrDefault(i => i.itemId == itemId);
        if (stack == null || stack.quantity < quantity) return false;
        
        stack.quantity -= quantity;
        if (stack.quantity <= 0)
            _items.Remove(stack);
        
        OnChanged?.Invoke();
        return true;
    }
}
```

---

## Save / Load

```csharp
[System.Serializable]
public class SaveData
{
    public Vector3 playerPosition;
    public int playerHealth;
    public List<ItemStack> inventory;
    public List<string> completedQuests;
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
    
    public static void Delete(string slot)
    {
        string path = GetPath(slot);
        if (File.Exists(path))
            File.Delete(path);
    }
    
    public static bool Exists(string slot) => File.Exists(GetPath(slot));
}

// Usage
public void SaveGame()
{
    var data = new SaveData
    {
        playerPosition = _player.transform.position,
        playerHealth = _player.Health,
        inventory = _inventory.Items.ToList(),
        completedQuests = _questManager.CompletedIds.ToList()
    };
    SaveSystem.Save(data, "slot1");
}

public void LoadGame()
{
    var data = SaveSystem.Load("slot1");
    if (data == null) return;
    
    _player.transform.position = data.playerPosition;
    _player.SetHealth(data.playerHealth);
    // Restore inventory, quests, etc.
}
```

---

## State Machine

```csharp
public interface IState
{
    void Enter();
    void Update();
    void Exit();
}

public class StateMachine
{
    private IState _currentState;
    
    public void ChangeState(IState newState)
    {
        _currentState?.Exit();
        _currentState = newState;
        _currentState?.Enter();
    }
    
    public void Update() => _currentState?.Update();
}

// Example states
public class IdleState : IState
{
    private readonly Enemy _enemy;
    
    public IdleState(Enemy enemy) => _enemy = enemy;
    
    public void Enter() => _enemy.PlayAnimation("Idle");
    public void Update()
    {
        if (_enemy.CanSeePlayer())
            _enemy.StateMachine.ChangeState(new ChaseState(_enemy));
    }
    public void Exit() { }
}

public class ChaseState : IState
{
    private readonly Enemy _enemy;
    
    public ChaseState(Enemy enemy) => _enemy = enemy;
    
    public void Enter() => _enemy.PlayAnimation("Run");
    public void Update() => _enemy.MoveToward(_enemy.Player.position);
    public void Exit() { }
}
```

---

## Timer

```csharp
public class Timer
{
    public float Duration { get; }
    public float Remaining { get; private set; }
    public bool IsComplete => Remaining <= 0;
    public float Progress => 1 - (Remaining / Duration);
    
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
}

// Usage
private Timer _attackCooldown;

private void Awake()
{
    _attackCooldown = new Timer(1f);
}

private void Update()
{
    _attackCooldown.Tick(Time.deltaTime);
    
    if (Input.GetButtonDown("Fire") && _attackCooldown.IsComplete)
    {
        Attack();
        _attackCooldown.Reset();
    }
}
```

---

## Coroutine Utilities

```csharp
// Delay
IEnumerator DelayedAction()
{
    yield return new WaitForSeconds(2f);
    DoSomething();
}

// Sequence
IEnumerator Sequence()
{
    yield return StartCoroutine(Step1());
    yield return StartCoroutine(Step2());
    yield return StartCoroutine(Step3());
}

// Lerp over time
IEnumerator MoveTo(Vector3 target, float duration)
{
    Vector3 start = transform.position;
    float elapsed = 0;
    
    while (elapsed < duration)
    {
        elapsed += Time.deltaTime;
        float t = elapsed / duration;
        transform.position = Vector3.Lerp(start, target, t);
        yield return null;
    }
    
    transform.position = target;
}

// Fade
IEnumerator Fade(CanvasGroup group, float from, float to, float duration)
{
    float elapsed = 0;
    
    while (elapsed < duration)
    {
        elapsed += Time.deltaTime;
        group.alpha = Mathf.Lerp(from, to, elapsed / duration);
        yield return null;
    }
    
    group.alpha = to;
}
```

---

## Simple Health Component

```csharp
public class Health : MonoBehaviour
{
    [SerializeField] private int _max = 100;
    
    public int Current { get; private set; }
    public int Max => _max;
    public bool IsDead => Current <= 0;
    public float Percentage => (float)Current / _max;
    
    public event Action<int, int> OnChanged;  // current, max
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
    
    public void SetMax(int max, bool healToFull = false)
    {
        _max = max;
        if (healToFull) Current = max;
        OnChanged?.Invoke(Current, _max);
    }
}
```

---

## Interaction System

```csharp
public interface IInteractable
{
    string GetPrompt();
    void Interact(GameObject interactor);
}

public class Interactor : MonoBehaviour
{
    [SerializeField] private float _range = 2f;
    [SerializeField] private LayerMask _interactableMask;
    
    private IInteractable _current;
    
    public IInteractable Current => _current;
    public event Action<IInteractable> OnTargetChanged;
    
    private void Update()
    {
        // Find interactable
        IInteractable found = null;
        if (Physics.Raycast(transform.position, transform.forward, out var hit, _range, _interactableMask))
        {
            found = hit.collider.GetComponent<IInteractable>();
        }
        
        if (found != _current)
        {
            _current = found;
            OnTargetChanged?.Invoke(_current);
        }
        
        // Interact
        if (_current != null && Input.GetKeyDown(KeyCode.E))
        {
            _current.Interact(gameObject);
        }
    }
}

// Example interactable
public class Door : MonoBehaviour, IInteractable
{
    private bool _isOpen;
    
    public string GetPrompt() => _isOpen ? "Close Door" : "Open Door";
    
    public void Interact(GameObject interactor)
    {
        _isOpen = !_isOpen;
        // Play animation, etc.
    }
}
```

---

## Camera Follow

```csharp
public class CameraFollow : MonoBehaviour
{
    [SerializeField] private Transform _target;
    [SerializeField] private Vector3 _offset = new Vector3(0, 5, -10);
    [SerializeField] private float _smoothSpeed = 5f;
    
    private void LateUpdate()
    {
        if (_target == null) return;
        
        Vector3 desiredPosition = _target.position + _offset;
        transform.position = Vector3.Lerp(transform.position, desiredPosition, _smoothSpeed * Time.deltaTime);
        transform.LookAt(_target);
    }
    
    public void SetTarget(Transform target) => _target = target;
}
```

---

## Audio One-Shot

```csharp
public class AudioManager : MonoBehaviour
{
    public static AudioManager Instance { get; private set; }
    
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
}
```

---

## Simple Tweening

```csharp
public static class Tween
{
    public static IEnumerator Move(Transform t, Vector3 to, float duration, System.Func<float, float> ease = null)
    {
        ease ??= (x) => x;  // Linear default
        Vector3 from = t.position;
        float elapsed = 0;
        
        while (elapsed < duration)
        {
            elapsed += Time.deltaTime;
            float t01 = ease(Mathf.Clamp01(elapsed / duration));
            t.position = Vector3.LerpUnclamped(from, to, t01);
            yield return null;
        }
        
        t.position = to;
    }
    
    public static IEnumerator Scale(Transform t, Vector3 to, float duration)
    {
        Vector3 from = t.localScale;
        float elapsed = 0;
        
        while (elapsed < duration)
        {
            elapsed += Time.deltaTime;
            t.localScale = Vector3.Lerp(from, to, elapsed / duration);
            yield return null;
        }
        
        t.localScale = to;
    }
    
    // Common easing functions
    public static float EaseOutQuad(float t) => 1 - (1 - t) * (1 - t);
    public static float EaseInQuad(float t) => t * t;
    public static float EaseInOutQuad(float t) => t < 0.5f ? 2 * t * t : 1 - Mathf.Pow(-2 * t + 2, 2) / 2;
}

// Usage
StartCoroutine(Tween.Move(transform, targetPos, 0.5f, Tween.EaseOutQuad));
```

For complex tweening, use DOTween (third-party asset).