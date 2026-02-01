# Design Patterns for C# Game Developers

---

# Behavioral Patterns

## Strategy Pattern

Define a family of algorithms and swap them at runtime without changing the client code.

**Use Cases:** AI behaviors, movement systems, attack types, damage calculation

```csharp
public interface IMovementStrategy
{
    void Move(Transform transform, float speed);
}

public class WalkMovement : IMovementStrategy
{
    public void Move(Transform transform, float speed)
    {
        transform.Translate(Vector3.forward * speed * Time.deltaTime);
    }
}

public class FlyMovement : IMovementStrategy
{
    public void Move(Transform transform, float speed)
    {
        transform.Translate(Vector3.up * speed * Time.deltaTime);
    }
}

public class PlayerController : MonoBehaviour
{
    private IMovementStrategy _movementStrategy;
    [SerializeField] private float _speed = 5f;

    private void Awake()
    {
        _movementStrategy = new WalkMovement();
    }

    public void SetMovementStrategy(IMovementStrategy strategy)
    {
        _movementStrategy = strategy;
    }

    private void Update()
    {
        _movementStrategy.Move(transform, _speed);
    }
}
```

---

## State Pattern

Allow an object to alter its behavior when its internal state changes. The object will appear to change its class.

**Use Cases:** Player states (idle, running, jumping), enemy AI states, game flow (menu, playing, paused)

```csharp
public interface IPlayerState
{
    void Enter(PlayerStateMachine player);
    void Update(PlayerStateMachine player);
    void Exit(PlayerStateMachine player);
}

public class IdleState : IPlayerState
{
    public void Enter(PlayerStateMachine player) 
    {
        player.Animator.Play("Idle");
    }

    public void Update(PlayerStateMachine player)
    {
        if (Input.GetAxis("Horizontal") != 0)
        {
            player.ChangeState(new RunningState());
        }
        if (Input.GetButtonDown("Jump"))
        {
            player.ChangeState(new JumpingState());
        }
    }

    public void Exit(PlayerStateMachine player) { }
}

public class RunningState : IPlayerState
{
    public void Enter(PlayerStateMachine player)
    {
        player.Animator.Play("Run");
    }

    public void Update(PlayerStateMachine player)
    {
        float horizontal = Input.GetAxis("Horizontal");
        player.transform.Translate(Vector3.right * horizontal * player.Speed * Time.deltaTime);
        
        if (horizontal == 0)
        {
            player.ChangeState(new IdleState());
        }
    }

    public void Exit(PlayerStateMachine player) { }
}

public class JumpingState : IPlayerState
{
    public void Enter(PlayerStateMachine player)
    {
        player.Animator.Play("Jump");
        player.Rigidbody.AddForce(Vector3.up * player.JumpForce, ForceMode.Impulse);
    }

    public void Update(PlayerStateMachine player)
    {
        if (player.IsGrounded)
        {
            player.ChangeState(new IdleState());
        }
    }

    public void Exit(PlayerStateMachine player) { }
}

public class PlayerStateMachine : MonoBehaviour
{
    public Animator Animator { get; private set; }
    public Rigidbody Rigidbody { get; private set; }
    public float Speed = 5f;
    public float JumpForce = 10f;
    public bool IsGrounded { get; private set; }

    private IPlayerState _currentState;

    private void Awake()
    {
        Animator = GetComponent<Animator>();
        Rigidbody = GetComponent<Rigidbody>();
        _currentState = new IdleState();
        _currentState.Enter(this);
    }

    public void ChangeState(IPlayerState newState)
    {
        _currentState.Exit(this);
        _currentState = newState;
        _currentState.Enter(this);
    }

    private void Update()
    {
        _currentState.Update(this);
    }
}
```

---

## Observer Pattern

Define a one-to-many dependency so that when one object changes state, all dependents are notified automatically.

**Use Cases:** Event systems, UI updates, achievement tracking, score changes

```csharp
// Using C# events (preferred)
public class Health : MonoBehaviour
{
    public event Action<int, int> OnHealthChanged; // current, max
    public event Action OnDeath;

    [SerializeField] private int _maxHealth = 100;
    private int _currentHealth;

    private void Awake()
    {
        _currentHealth = _maxHealth;
    }

    public void TakeDamage(int amount)
    {
        _currentHealth = Mathf.Max(0, _currentHealth - amount);
        OnHealthChanged?.Invoke(_currentHealth, _maxHealth);

        if (_currentHealth <= 0)
        {
            OnDeath?.Invoke();
        }
    }

    public void Heal(int amount)
    {
        _currentHealth = Mathf.Min(_maxHealth, _currentHealth + amount);
        OnHealthChanged?.Invoke(_currentHealth, _maxHealth);
    }
}

public class HealthUI : MonoBehaviour
{
    [SerializeField] private Health _playerHealth;
    [SerializeField] private Slider _healthBar;

    private void OnEnable()
    {
        _playerHealth.OnHealthChanged += UpdateHealthBar;
        _playerHealth.OnDeath += HandleDeath;
    }

    private void OnDisable()
    {
        _playerHealth.OnHealthChanged -= UpdateHealthBar;
        _playerHealth.OnDeath -= HandleDeath;
    }

    private void UpdateHealthBar(int current, int max)
    {
        _healthBar.value = (float)current / max;
    }

    private void HandleDeath()
    {
        Debug.Log("Player died!");
    }
}

// Alternative: ScriptableObject-based events (decoupled, designer-friendly)
[CreateAssetMenu(menuName = "Events/Game Event")]
public class GameEvent : ScriptableObject
{
    private readonly List<Action> _listeners = new();

    public void Raise()
    {
        for (int i = _listeners.Count - 1; i >= 0; i--)
        {
            _listeners[i]?.Invoke();
        }
    }

    public void RegisterListener(Action listener) => _listeners.Add(listener);
    public void UnregisterListener(Action listener) => _listeners.Remove(listener);
}
```

---

## Command Pattern

Encapsulate a request as an object, allowing you to parameterize, queue, or undo operations.

**Use Cases:** Input systems, undo/redo, replay systems, action queues, turn-based games

```csharp
public interface ICommand
{
    void Execute();
    void Undo();
}

public class MoveCommand : ICommand
{
    private readonly Transform _transform;
    private readonly Vector3 _direction;
    private readonly float _distance;
    private Vector3 _previousPosition;

    public MoveCommand(Transform transform, Vector3 direction, float distance)
    {
        _transform = transform;
        _direction = direction;
        _distance = distance;
    }

    public void Execute()
    {
        _previousPosition = _transform.position;
        _transform.position += _direction * _distance;
    }

    public void Undo()
    {
        _transform.position = _previousPosition;
    }
}

public class CommandInvoker
{
    private readonly Stack<ICommand> _undoStack = new();
    private readonly Stack<ICommand> _redoStack = new();

    public void ExecuteCommand(ICommand command)
    {
        command.Execute();
        _undoStack.Push(command);
        _redoStack.Clear();
    }

    public void Undo()
    {
        if (_undoStack.Count == 0) return;
        
        var command = _undoStack.Pop();
        command.Undo();
        _redoStack.Push(command);
    }

    public void Redo()
    {
        if (_redoStack.Count == 0) return;
        
        var command = _redoStack.Pop();
        command.Execute();
        _undoStack.Push(command);
    }
}

public class PlayerInput : MonoBehaviour
{
    private CommandInvoker _invoker = new();
    [SerializeField] private float _moveDistance = 1f;

    private void Update()
    {
        if (Input.GetKeyDown(KeyCode.W))
            _invoker.ExecuteCommand(new MoveCommand(transform, Vector3.forward, _moveDistance));
        if (Input.GetKeyDown(KeyCode.S))
            _invoker.ExecuteCommand(new MoveCommand(transform, Vector3.back, _moveDistance));
        if (Input.GetKeyDown(KeyCode.Z))
            _invoker.Undo();
        if (Input.GetKeyDown(KeyCode.Y))
            _invoker.Redo();
    }
}
```

---

## Subclass Sandbox Pattern

Define behavior in a base class using abstract operations that subclasses implement. The base class provides protected methods that subclasses can use to compose their behavior.

**Use Cases:** Spell/ability systems, power-ups, enemy behaviors with shared utilities

```csharp
public abstract class Superpower : MonoBehaviour
{
    // Sandbox methods - protected utilities for subclasses
    protected void PlaySound(AudioClip clip, float volume = 1f)
    {
        AudioSource.PlayClipAtPoint(clip, transform.position, volume);
    }

    protected void SpawnParticles(ParticleSystem prefab, Vector3 position)
    {
        Instantiate(prefab, position, Quaternion.identity);
    }

    protected GameObject SpawnProjectile(GameObject prefab, Vector3 position, Vector3 direction)
    {
        var projectile = Instantiate(prefab, position, Quaternion.LookRotation(direction));
        return projectile;
    }

    protected IEnumerable<Collider> GetEnemiesInRange(float radius)
    {
        return Physics.OverlapSphere(transform.position, radius)
            .Where(c => c.CompareTag("Enemy"));
    }

    protected void DealDamage(Health target, int amount)
    {
        target.TakeDamage(amount);
    }

    // Abstract method that subclasses must implement
    public abstract void Activate();
}

// Subclasses compose behavior using sandbox methods
public class FireballPower : Superpower
{
    [SerializeField] private GameObject _fireballPrefab;
    [SerializeField] private ParticleSystem _castEffect;
    [SerializeField] private AudioClip _castSound;

    public override void Activate()
    {
        PlaySound(_castSound);
        SpawnParticles(_castEffect, transform.position);
        SpawnProjectile(_fireballPrefab, transform.position, transform.forward);
    }
}

public class ShockwavePower : Superpower
{
    [SerializeField] private float _radius = 5f;
    [SerializeField] private int _damage = 20;
    [SerializeField] private ParticleSystem _shockwaveEffect;
    [SerializeField] private AudioClip _shockwaveSound;

    public override void Activate()
    {
        PlaySound(_shockwaveSound, 1.5f);
        SpawnParticles(_shockwaveEffect, transform.position);

        foreach (var enemy in GetEnemiesInRange(_radius))
        {
            var health = enemy.GetComponent<Health>();
            if (health != null)
            {
                DealDamage(health, _damage);
            }
        }
    }
}
```

**Why use this?** Without the sandbox, every subclass would need to know about AudioSource, Instantiate, Physics queries, etc. The sandbox provides a clean API that keeps subclasses focused on their unique behavior.

---

## Bytecode Pattern

Create a simple virtual machine with instructions that can be composed to define behavior. Lets designers or modders create complex behaviors without touching game code.

**Use Cases:** Spell systems, AI scripting, modding support, cutscene scripting

```csharp
public enum Instruction
{
    // Stack operations
    Literal,        // Push a number onto the stack
    
    // Actions
    PlaySound,      // Pop sound ID, play it
    SpawnParticles, // Pop particle ID, spawn at caster
    GetHealth,      // Push caster's health
    SetHealth,      // Pop value, set caster's health
    GetTarget,      // Push current target onto stack
    DealDamage,     // Pop amount, deal to target
    
    // Math
    Add,
    Multiply,
    
    // Control
    End
}

public class SpellVM
{
    private readonly Stack<int> _stack = new();
    private readonly GameObject _caster;
    private readonly GameObject _target;
    private readonly AudioClip[] _sounds;
    private readonly ParticleSystem[] _particles;

    public SpellVM(GameObject caster, GameObject target, AudioClip[] sounds, ParticleSystem[] particles)
    {
        _caster = caster;
        _target = target;
        _sounds = sounds;
        _particles = particles;
    }

    public void Execute(int[] bytecode)
    {
        int ip = 0; // instruction pointer
        
        while (ip < bytecode.Length)
        {
            var instruction = (Instruction)bytecode[ip++];
            
            switch (instruction)
            {
                case Instruction.Literal:
                    _stack.Push(bytecode[ip++]);
                    break;
                    
                case Instruction.PlaySound:
                    var soundId = _stack.Pop();
                    AudioSource.PlayClipAtPoint(_sounds[soundId], _caster.transform.position);
                    break;
                    
                case Instruction.SpawnParticles:
                    var particleId = _stack.Pop();
                    Object.Instantiate(_particles[particleId], _caster.transform.position, Quaternion.identity);
                    break;
                    
                case Instruction.GetHealth:
                    var health = _caster.GetComponent<Health>();
                    _stack.Push(health.Current);
                    break;
                    
                case Instruction.SetHealth:
                    var newHealth = _stack.Pop();
                    _caster.GetComponent<Health>().SetHealth(newHealth);
                    break;
                    
                case Instruction.DealDamage:
                    var damage = _stack.Pop();
                    _target.GetComponent<Health>().TakeDamage(damage);
                    break;
                    
                case Instruction.Add:
                    _stack.Push(_stack.Pop() + _stack.Pop());
                    break;
                    
                case Instruction.Multiply:
                    _stack.Push(_stack.Pop() * _stack.Pop());
                    break;
                    
                case Instruction.End:
                    return;
            }
        }
    }
}

// Usage - a "fireball" spell defined as bytecode
// This could be loaded from a file, letting designers create spells without code
public class SpellCaster : MonoBehaviour
{
    [SerializeField] private AudioClip[] _sounds;
    [SerializeField] private ParticleSystem[] _particles;

    // Fireball: play sound 0, spawn particles 1, deal 25 damage
    private readonly int[] _fireballSpell = {
        (int)Instruction.Literal, 0,
        (int)Instruction.PlaySound,
        (int)Instruction.Literal, 1,
        (int)Instruction.SpawnParticles,
        (int)Instruction.Literal, 25,
        (int)Instruction.DealDamage,
        (int)Instruction.End
    };

    public void CastFireball(GameObject target)
    {
        var vm = new SpellVM(gameObject, target, _sounds, _particles);
        vm.Execute(_fireballSpell);
    }
}
```

**Why use this?** Spells/abilities can be data-driven and loaded from files. Designers can create new spells without programmer involvement. Great for modding support.

---

# Creational Patterns

## Factory Pattern

Create objects without specifying the exact class. Centralizes creation logic.

**Use Cases:** Enemy spawning, weapon creation, UI element generation, power-up instantiation

```csharp
public interface IEnemy
{
    void Initialize();
    void Attack();
}

public class Zombie : MonoBehaviour, IEnemy
{
    public void Initialize() => Debug.Log("Zombie spawned");
    public void Attack() => Debug.Log("Zombie bites!");
}

public class Skeleton : MonoBehaviour, IEnemy
{
    public void Initialize() => Debug.Log("Skeleton spawned");
    public void Attack() => Debug.Log("Skeleton slashes!");
}

public class Ghost : MonoBehaviour, IEnemy
{
    public void Initialize() => Debug.Log("Ghost spawned");
    public void Attack() => Debug.Log("Ghost haunts!");
}

public enum EnemyType { Zombie, Skeleton, Ghost }

public class EnemyFactory : MonoBehaviour
{
    [SerializeField] private GameObject _zombiePrefab;
    [SerializeField] private GameObject _skeletonPrefab;
    [SerializeField] private GameObject _ghostPrefab;

    public IEnemy CreateEnemy(EnemyType type, Vector3 position)
    {
        GameObject prefab = type switch
        {
            EnemyType.Zombie => _zombiePrefab,
            EnemyType.Skeleton => _skeletonPrefab,
            EnemyType.Ghost => _ghostPrefab,
            _ => throw new ArgumentException($"Unknown enemy type: {type}")
        };

        var instance = Instantiate(prefab, position, Quaternion.identity);
        var enemy = instance.GetComponent<IEnemy>();
        enemy.Initialize();
        return enemy;
    }
}
```

---

## Abstract Factory Pattern

Create families of related objects without specifying their concrete classes.

**Use Cases:** Theme/skin systems, difficulty-based content, platform-specific implementations

```csharp
public interface IUIFactory
{
    IButton CreateButton();
    IPanel CreatePanel();
}

public interface IButton
{
    void Render();
}

public interface IPanel
{
    void Render();
}

// Fantasy theme
public class FantasyButton : IButton
{
    public void Render() => Debug.Log("Rendering ornate wooden button");
}

public class FantasyPanel : IPanel
{
    public void Render() => Debug.Log("Rendering parchment panel");
}

public class FantasyUIFactory : IUIFactory
{
    public IButton CreateButton() => new FantasyButton();
    public IPanel CreatePanel() => new FantasyPanel();
}

// Sci-fi theme
public class SciFiButton : IButton
{
    public void Render() => Debug.Log("Rendering holographic button");
}

public class SciFiPanel : IPanel
{
    public void Render() => Debug.Log("Rendering metal panel");
}

public class SciFiUIFactory : IUIFactory
{
    public IButton CreateButton() => new SciFiButton();
    public IPanel CreatePanel() => new SciFiPanel();
}

public class UIManager
{
    private readonly IUIFactory _factory;

    public UIManager(IUIFactory factory)
    {
        _factory = factory;
    }

    public void BuildUI()
    {
        var button = _factory.CreateButton();
        var panel = _factory.CreatePanel();
        button.Render();
        panel.Render();
    }
}
```

---

## Prototype Pattern

Create new objects by cloning existing ones rather than instantiating from scratch.

**Use Cases:** Spawning variations of enemies, duplicating items, runtime object creation from templates

```csharp
public interface IPrototype<T>
{
    T Clone();
}

public class Monster : IPrototype<Monster>
{
    public string Name { get; set; }
    public int Health { get; set; }
    public int Attack { get; set; }
    public float Speed { get; set; }

    public Monster Clone()
    {
        return new Monster
        {
            Name = Name,
            Health = Health,
            Attack = Attack,
            Speed = Speed
        };
    }
}

// Spawner that uses prototypes
public class MonsterSpawner
{
    private readonly Dictionary<string, Monster> _prototypes = new();

    public void RegisterPrototype(string name, Monster prototype)
    {
        _prototypes[name] = prototype;
    }

    public Monster Spawn(string prototypeName)
    {
        if (_prototypes.TryGetValue(prototypeName, out var prototype))
        {
            return prototype.Clone();
        }
        throw new ArgumentException($"Unknown prototype: {prototypeName}");
    }

    public Monster SpawnVariant(string prototypeName, Action<Monster> customize)
    {
        var monster = Spawn(prototypeName);
        customize(monster);
        return monster;
    }
}

// Usage
var spawner = new MonsterSpawner();

// Register base prototypes
spawner.RegisterPrototype("goblin", new Monster { Name = "Goblin", Health = 30, Attack = 5, Speed = 3f });
spawner.RegisterPrototype("orc", new Monster { Name = "Orc", Health = 100, Attack = 15, Speed = 2f });

// Spawn from prototypes
var goblin1 = spawner.Spawn("goblin");
var goblin2 = spawner.Spawn("goblin");  // Independent clone

// Spawn with variations
var eliteGoblin = spawner.SpawnVariant("goblin", m => {
    m.Name = "Elite Goblin";
    m.Health *= 2;
    m.Attack *= 2;
});
```

---

## Singleton Pattern

Ensure a class has only one instance and provide a global access point.

**Use Cases:** Game managers, audio managers, input managers

**Warning:** Overuse creates tight coupling and testing difficulties. Prefer dependency injection when possible.

```csharp
// Basic thread-safe singleton
public class GameManager : MonoBehaviour
{
    public static GameManager Instance { get; private set; }
    
    public int Score { get; private set; }

    private void Awake()
    {
        if (Instance != null && Instance != this)
        {
            Destroy(gameObject);
            return;
        }
        Instance = this;
        DontDestroyOnLoad(gameObject);
    }

    public void AddScore(int amount) => Score += amount;
}

// Generic singleton base class
public abstract class Singleton<T> : MonoBehaviour where T : MonoBehaviour
{
    private static T _instance;
    private static readonly object _lock = new();
    private static bool _applicationIsQuitting;

    public static T Instance
    {
        get
        {
            if (_applicationIsQuitting)
            {
                Debug.LogWarning($"[Singleton] Instance of {typeof(T)} already destroyed.");
                return null;
            }

            lock (_lock)
            {
                if (_instance == null)
                {
                    _instance = FindObjectOfType<T>();
                    
                    if (_instance == null)
                    {
                        var singleton = new GameObject($"[Singleton] {typeof(T)}");
                        _instance = singleton.AddComponent<T>();
                        DontDestroyOnLoad(singleton);
                    }
                }
                return _instance;
            }
        }
    }

    protected virtual void OnApplicationQuit()
    {
        _applicationIsQuitting = true;
    }
}

// Usage
public class AudioManager : Singleton<AudioManager>
{
    public void PlaySound(AudioClip clip) { /* ... */ }
}
```

---

## Object Pool Pattern

Reuse objects instead of creating/destroying them repeatedly. Critical for performance.

**Use Cases:** Bullets, particles, enemies, any frequently spawned objects

```csharp
public class ObjectPool<T> where T : MonoBehaviour
{
    private readonly T _prefab;
    private readonly Transform _parent;
    private readonly Queue<T> _pool = new();
    private readonly int _initialSize;

    public ObjectPool(T prefab, int initialSize, Transform parent = null)
    {
        _prefab = prefab;
        _initialSize = initialSize;
        _parent = parent;
        
        Prewarm();
    }

    private void Prewarm()
    {
        for (int i = 0; i < _initialSize; i++)
        {
            var obj = Object.Instantiate(_prefab, _parent);
            obj.gameObject.SetActive(false);
            _pool.Enqueue(obj);
        }
    }

    public T Get()
    {
        T obj = _pool.Count > 0 
            ? _pool.Dequeue() 
            : Object.Instantiate(_prefab, _parent);
        
        obj.gameObject.SetActive(true);
        return obj;
    }

    public void Return(T obj)
    {
        obj.gameObject.SetActive(false);
        _pool.Enqueue(obj);
    }
}

// Usage with bullets
public class BulletPool : MonoBehaviour
{
    [SerializeField] private Bullet _bulletPrefab;
    [SerializeField] private int _poolSize = 50;
    
    private ObjectPool<Bullet> _pool;

    private void Awake()
    {
        _pool = new ObjectPool<Bullet>(_bulletPrefab, _poolSize, transform);
    }

    public Bullet GetBullet() => _pool.Get();
    public void ReturnBullet(Bullet bullet) => _pool.Return(bullet);
}

public class Bullet : MonoBehaviour
{
    private BulletPool _pool;
    
    public void Initialize(BulletPool pool) => _pool = pool;
    
    private void OnCollisionEnter(Collision collision)
    {
        _pool.ReturnBullet(this);
    }
}
```

---

## Builder Pattern

Construct complex objects step by step. Separates construction from representation.

**Use Cases:** Character creation, level generation, complex UI construction

```csharp
public class Character
{
    public string Name { get; set; }
    public int Health { get; set; }
    public int Strength { get; set; }
    public int Intelligence { get; set; }
    public List<string> Abilities { get; set; } = new();
    public GameObject Model { get; set; }
}

public class CharacterBuilder
{
    private readonly Character _character = new();

    public CharacterBuilder WithName(string name)
    {
        _character.Name = name;
        return this;
    }

    public CharacterBuilder WithHealth(int health)
    {
        _character.Health = health;
        return this;
    }

    public CharacterBuilder WithStrength(int strength)
    {
        _character.Strength = strength;
        return this;
    }

    public CharacterBuilder WithIntelligence(int intelligence)
    {
        _character.Intelligence = intelligence;
        return this;
    }

    public CharacterBuilder WithAbility(string ability)
    {
        _character.Abilities.Add(ability);
        return this;
    }

    public CharacterBuilder WithModel(GameObject model)
    {
        _character.Model = Object.Instantiate(model);
        return this;
    }

    public Character Build() => _character;
}

// Usage
public class CharacterCreator : MonoBehaviour
{
    [SerializeField] private GameObject _warriorModel;

    private void Start()
    {
        var warrior = new CharacterBuilder()
            .WithName("Conan")
            .WithHealth(150)
            .WithStrength(20)
            .WithIntelligence(8)
            .WithAbility("Slash")
            .WithAbility("Shield Block")
            .WithModel(_warriorModel)
            .Build();
    }
}
```

---

# Structural Patterns

## Component Pattern

Compose objects from smaller, reusable components. **Unity's core architecture.**

**Use Cases:** Everything in Unity—this is the engine's fundamental design

```csharp
// Unity already implements this - embrace it
public class HealthComponent : MonoBehaviour
{
    [SerializeField] private int _maxHealth = 100;
    private int _currentHealth;
    
    public event Action<int> OnDamaged;
    public event Action OnDeath;

    private void Awake() => _currentHealth = _maxHealth;
    
    public void TakeDamage(int amount)
    {
        _currentHealth -= amount;
        OnDamaged?.Invoke(amount);
        if (_currentHealth <= 0) OnDeath?.Invoke();
    }
}

public class MovementComponent : MonoBehaviour
{
    [SerializeField] private float _speed = 5f;
    
    public void Move(Vector3 direction)
    {
        transform.Translate(direction * _speed * Time.deltaTime);
    }
}

public class AttackComponent : MonoBehaviour
{
    [SerializeField] private int _damage = 10;
    [SerializeField] private float _range = 2f;

    public void Attack(HealthComponent target)
    {
        if (Vector3.Distance(transform.position, target.transform.position) <= _range)
        {
            target.TakeDamage(_damage);
        }
    }
}

// Compose entities from components
// Player: HealthComponent + MovementComponent + AttackComponent
// Enemy:  HealthComponent + MovementComponent + AttackComponent + AIComponent
// Turret: HealthComponent + AttackComponent (no movement)
```

---

## Decorator Pattern

Attach additional responsibilities to objects dynamically without subclassing.

**Use Cases:** Weapon modifiers, buff/debuff systems, ability enhancements

```csharp
public interface IWeapon
{
    int Damage { get; }
    string Description { get; }
    void Attack();
}

public class Sword : IWeapon
{
    public int Damage => 10;
    public string Description => "Sword";
    public void Attack() => Debug.Log($"Slash for {Damage} damage");
}

public abstract class WeaponDecorator : IWeapon
{
    protected readonly IWeapon _weapon;  // 1. HOLDS a weapon
    
    protected WeaponDecorator(IWeapon weapon) => _weapon = weapon;  // 2. Receives it
    
    public virtual int Damage => _weapon.Damage;  // 3. DELEGATES to it
    public virtual string Description => _weapon.Description;
    public virtual void Attack() => _weapon.Attack();
}

public class FireEnchantment : WeaponDecorator
{
    public FireEnchantment(IWeapon weapon) : base(weapon) { }
    
    public override int Damage => _weapon.Damage + 5;
    public override string Description => $"Flaming {_weapon.Description}";
    public override void Attack()
    {
        base.Attack();
        Debug.Log("Burns for 5 extra fire damage!");
    }
}

public class PoisonEnchantment : WeaponDecorator
{
    public PoisonEnchantment(IWeapon weapon) : base(weapon) { }
    
    public override int Damage => _weapon.Damage + 3;
    public override string Description => $"Poisoned {_weapon.Description}";
    public override void Attack()
    {
        base.Attack();
        Debug.Log("Applies poison for 3 damage over time!");
    }
}

// Usage: Stack decorators
IWeapon weapon = new Sword();                           // 10 damage
weapon = new FireEnchantment(weapon);                   // Wraps Sword 15 damage
weapon = new PoisonEnchantment(weapon);                 // Wraps Fire(Sword) 18 damage
Debug.Log(weapon.Description);                          // "Poisoned Flaming Sword"
```

---

## Flyweight Pattern

Share common state between many objects to minimize memory usage.

**Use Cases:** Terrain tiles, tree/foliage rendering, particle systems, bullet types

```csharp
// Shared data (intrinsic state) - flyweight
[CreateAssetMenu(menuName = "Game/Tree Type")]
public class TreeType : ScriptableObject
{
    public string treeName;
    public Mesh mesh;
    public Material material;
    public float baseScale;
}

// Per-instance data (extrinsic state)
public class TreeInstance
{
    public Vector3 Position { get; }
    public float Scale { get; }
    public float Rotation { get; }
    public TreeType Type { get; }  // Reference to shared flyweight

    public TreeInstance(Vector3 position, float scale, float rotation, TreeType type)
    {
        Position = position;
        Scale = scale;
        Rotation = rotation;
        Type = type;
    }

    public void Render()
    {
        var matrix = Matrix4x4.TRS(
            Position,
            Quaternion.Euler(0, Rotation, 0),
            Vector3.one * Type.baseScale * Scale
        );
        Graphics.DrawMesh(Type.mesh, matrix, Type.material, 0);
    }
}

// Forest with thousands of trees but only a few TreeType objects
public class Forest : MonoBehaviour
{
    [SerializeField] private TreeType[] _treeTypes;
    private readonly List<TreeInstance> _trees = new();

    public void PlantTree(Vector3 position, int typeIndex)
    {
        var tree = new TreeInstance(
            position,
            Random.Range(0.8f, 1.2f),
            Random.Range(0f, 360f),
            _treeTypes[typeIndex]  // Shared reference
        );
        _trees.Add(tree);
    }

    private void Update()
    {
        foreach (var tree in _trees)
        {
            tree.Render();
        }
    }
}

// 10,000 trees but only 5 TreeType objects in memory
```

---

## Facade Pattern

Provide a simplified interface to a complex subsystem.

**Use Cases:** Complex API wrappers, save system interfaces, audio management

```csharp
// Complex subsystems
public class SaveSystem
{
    public void SaveToFile(string data, string path) { /* ... */ }
    public string LoadFromFile(string path) { /* ... */ return ""; }
}

public class EncryptionSystem
{
    public string Encrypt(string data) { /* ... */ return data; }
    public string Decrypt(string data) { /* ... */ return data; }
}

public class CloudSyncSystem
{
    public void Upload(string data) { /* ... */ }
    public string Download() { /* ... */ return ""; }
}

public class SerializationSystem
{
    public string Serialize<T>(T obj) => JsonUtility.ToJson(obj);
    public T Deserialize<T>(string json) => JsonUtility.FromJson<T>(json);
}

// Facade
public class SaveManager
{
    private readonly SaveSystem _saveSystem = new();
    private readonly EncryptionSystem _encryption = new();
    private readonly CloudSyncSystem _cloudSync = new();
    private readonly SerializationSystem _serialization = new();
    
    public void SaveGame<T>(T gameState, string slot)
    {
        string json = _serialization.Serialize(gameState);
        string encrypted = _encryption.Encrypt(json);
        _saveSystem.SaveToFile(encrypted, $"save_{slot}.dat");
        _cloudSync.Upload(encrypted);
    }
    
    public T LoadGame<T>(string slot)
    {
        string encrypted = _saveSystem.LoadFromFile($"save_{slot}.dat");
        string json = _encryption.Decrypt(encrypted);
        return _serialization.Deserialize<T>(json);
    }
}

// Client code is simple
var saveManager = new SaveManager();
saveManager.SaveGame(playerData, "slot1");
var loaded = saveManager.LoadGame<PlayerData>("slot1");
```

---

## Service Locator Pattern

Provide a global point of access to services without hard-coding dependencies.

**Use Cases:** Accessing game systems, audio, input, analytics

**Note:** Prefer dependency injection when possible; service locator is a controlled form of global state.

```csharp
public interface IAudioService
{
    void PlaySound(string soundName);
    void PlayMusic(string trackName);
}

public interface IInputService
{
    Vector2 GetMovementInput();
    bool IsJumpPressed();
}

public static class ServiceLocator
{
    private static readonly Dictionary<Type, object> _services = new();

    public static void Register<T>(T service) where T : class
    {
        _services[typeof(T)] = service;
    }

    public static T Get<T>() where T : class
    {
        if (_services.TryGetValue(typeof(T), out var service))
        {
            return service as T;
        }
        throw new InvalidOperationException($"Service {typeof(T)} not registered");
    }

    public static void Clear() => _services.Clear();
}

// Registration (at game startup)
public class GameBootstrap : MonoBehaviour
{
    [SerializeField] private AudioService _audioService;
    [SerializeField] private InputService _inputService;

    private void Awake()
    {
        ServiceLocator.Register<IAudioService>(_audioService);
        ServiceLocator.Register<IInputService>(_inputService);
    }
}

// Usage anywhere
public class Player : MonoBehaviour
{
    private void Update()
    {
        var input = ServiceLocator.Get<IInputService>();
        Vector2 movement = input.GetMovementInput();
        
        if (input.IsJumpPressed())
        {
            ServiceLocator.Get<IAudioService>().PlaySound("jump");
        }
    }
}
```

---

# Sequencing Patterns

## Game Loop Pattern

The heartbeat of every game. Processes input, updates game state, and renders—repeatedly, as fast as possible (or at a fixed rate).

**Note:** Unity/Godot/Unreal handle this for you, but understanding it helps you work with the engine better.

```csharp
// Conceptual game loop (what the engine does internally)
public class GameLoop
{
    private bool _isRunning = true;
    private double _previousTime;
    private double _lag;
    private const double MS_PER_UPDATE = 16.67; // ~60 FPS for physics

    public void Run()
    {
        _previousTime = GetCurrentTimeMs();

        while (_isRunning)
        {
            double currentTime = GetCurrentTimeMs();
            double elapsed = currentTime - _previousTime;
            _previousTime = currentTime;
            _lag += elapsed;

            ProcessInput();

            // Fixed timestep for physics/game logic
            while (_lag >= MS_PER_UPDATE)
            {
                Update(MS_PER_UPDATE / 1000.0);
                _lag -= MS_PER_UPDATE;
            }

            // Render with interpolation for smooth visuals
            Render(_lag / MS_PER_UPDATE);
        }
    }

    private void ProcessInput() { /* Poll input devices */ }
    private void Update(double deltaTime) { /* Physics, AI, game logic */ }
    private void Render(double interpolation) { /* Draw everything */ }
    private double GetCurrentTimeMs() => DateTime.Now.Ticks / TimeSpan.TicksPerMillisecond;
}

// In Unity, the engine handles this. You just use:
// - FixedUpdate() for physics (fixed timestep)
// - Update() for game logic (variable timestep)
// - LateUpdate() for camera follow, etc.
public class UnityGameLoop : MonoBehaviour
{
    private void FixedUpdate()
    {
        // Physics, consistent game logic
        // Called at fixed intervals (default 50 times/sec)
    }

    private void Update()
    {
        // Input, variable game logic
        // Called once per frame (variable rate)
    }

    private void LateUpdate()
    {
        // Camera follow, post-processing
        // Called after all Update() calls
    }
}
```

**Key insight:** The loop separates _simulation rate_ (fixed, for deterministic physics) from _render rate_ (variable, for smooth visuals).

---

## Update Method Pattern

Each game entity has an Update() method called once per frame. Entities manage their own behavior.

**Note:** Unity does this automatically with MonoBehaviour.Update(). Understanding why helps you use it better.

```csharp
// The pattern (what Unity does under the hood)
public abstract class Entity
{
    public bool IsActive { get; set; } = true;
    public abstract void Update(float deltaTime);
}

public class GameWorld
{
    private readonly List<Entity> _entities = new();

    public void Add(Entity entity) => _entities.Add(entity);
    public void Remove(Entity entity) => _entities.Remove(entity);

    public void UpdateAll(float deltaTime)
    {
        // Copy list to allow modification during iteration
        foreach (var entity in _entities.ToList())
        {
            if (entity.IsActive)
            {
                entity.Update(deltaTime);
            }
        }
    }
}

// Entities define their own behavior
public class Skeleton : Entity
{
    private Vector3 _position;
    private float _patrolTimer;

    public override void Update(float deltaTime)
    {
        _patrolTimer += deltaTime;
        if (_patrolTimer > 2f)
        {
            // Change direction
            _patrolTimer = 0;
        }
        // Move, animate, etc.
    }
}

// In Unity, just use MonoBehaviour
public class SkeletonEnemy : MonoBehaviour
{
    private float _patrolTimer;

    private void Update()  // Unity calls this automatically
    {
        _patrolTimer += Time.deltaTime;
        if (_patrolTimer > 2f)
        {
            // Change direction
            _patrolTimer = 0;
        }
    }
}
```

**Gotcha:** Be careful with Update() order. If entity A needs to read entity B's position after B moves, you might get stale data. Use `LateUpdate()` or explicit ordering when it matters.

---

## Double Buffer Pattern

Maintain two copies of state: one being read (current) while the other is written (next). Swap when done. Prevents readers from seeing incomplete state.

**Use Cases:** Rendering (front/back buffer), physics state, turn-based simultaneous actions

```csharp
// Generic double buffer
public class DoubleBuffer<T> where T : new()
{
    private T _current = new();
    private T _next = new();

    public T Current => _current;
    public T Next => _next;

    public void Swap()
    {
        (_current, _next) = (_next, _current);
    }
}

// Example: Grid-based game where all cells update simultaneously
public class CellState
{
    public bool IsAlive;
}

public class GameOfLife : MonoBehaviour
{
    private const int Width = 50;
    private const int Height = 50;
    
    private CellState[,] _currentGrid = new CellState[Width, Height];
    private CellState[,] _nextGrid = new CellState[Width, Height];

    private void Start()
    {
        // Initialize grids
        for (int x = 0; x < Width; x++)
        {
            for (int y = 0; y < Height; y++)
            {
                _currentGrid[x, y] = new CellState { IsAlive = Random.value > 0.5f };
                _nextGrid[x, y] = new CellState();
            }
        }
    }

    private void Update()
    {
        // Calculate next state based on current (read from current, write to next)
        for (int x = 0; x < Width; x++)
        {
            for (int y = 0; y < Height; y++)
            {
                int neighbors = CountAliveNeighbors(x, y);
                bool currentlyAlive = _currentGrid[x, y].IsAlive;

                // Conway's rules
                _nextGrid[x, y].IsAlive = currentlyAlive
                    ? neighbors == 2 || neighbors == 3
                    : neighbors == 3;
            }
        }

        // Swap buffers
        (_currentGrid, _nextGrid) = (_nextGrid, _currentGrid);
    }

    private int CountAliveNeighbors(int x, int y)
    {
        int count = 0;
        for (int dx = -1; dx <= 1; dx++)
        {
            for (int dy = -1; dy <= 1; dy++)
            {
                if (dx == 0 && dy == 0) continue;
                int nx = (x + dx + Width) % Width;
                int ny = (y + dy + Height) % Height;
                if (_currentGrid[nx, ny].IsAlive) count++;
            }
        }
        return count;
    }
}
```

**Why?** Without double buffering, a cell's update might see some neighbors in their old state and others in their new state—inconsistent and wrong.

---

# Decoupling Patterns

## Event Queue Pattern

Decouple event senders from receivers using a queue. Events are sent to the queue and processed later.

**Use Cases:** Audio system, achievements, analytics, network messages, deferred actions

```csharp
public struct GameEvent
{
    public enum Type { EnemyKilled, ItemCollected, LevelCompleted, DamageTaken }
    
    public Type EventType;
    public object Data;
}

public class EventQueue
{
    private readonly Queue<GameEvent> _queue = new();
    private readonly Dictionary<GameEvent.Type, List<Action<object>>> _listeners = new();
    private const int MaxEventsPerFrame = 10;

    public void Subscribe(GameEvent.Type type, Action<object> handler)
    {
        if (!_listeners.ContainsKey(type))
            _listeners[type] = new List<Action<object>>();
        _listeners[type].Add(handler);
    }

    public void Unsubscribe(GameEvent.Type type, Action<object> handler)
    {
        if (_listeners.ContainsKey(type))
            _listeners[type].Remove(handler);
    }

    public void Enqueue(GameEvent.Type type, object data = null)
    {
        _queue.Enqueue(new GameEvent { EventType = type, Data = data });
    }

    // Call this once per frame
    public void ProcessEvents()
    {
        int processed = 0;
        while (_queue.Count > 0 && processed < MaxEventsPerFrame)
        {
            var evt = _queue.Dequeue();
            
            if (_listeners.TryGetValue(evt.EventType, out var handlers))
            {
                foreach (var handler in handlers)
                {
                    handler(evt.Data);
                }
            }
            processed++;
        }
    }
}

// Usage
public class GameEventManager : MonoBehaviour
{
    public static EventQueue Events { get; } = new();

    private void Update()
    {
        Events.ProcessEvents();
    }
}

// Sender (doesn't know about receivers)
public class Enemy : MonoBehaviour
{
    private void Die()
    {
        GameEventManager.Events.Enqueue(GameEvent.Type.EnemyKilled, this);
        Destroy(gameObject);
    }
}

// Receivers (don't know about senders)
public class ScoreManager : MonoBehaviour
{
    private void OnEnable()
    {
        GameEventManager.Events.Subscribe(GameEvent.Type.EnemyKilled, OnEnemyKilled);
    }

    private void OnDisable()
    {
        GameEventManager.Events.Unsubscribe(GameEvent.Type.EnemyKilled, OnEnemyKilled);
    }

    private void OnEnemyKilled(object data) => AddScore(100);
}

public class AudioManager : MonoBehaviour
{
    private void OnEnable()
    {
        GameEventManager.Events.Subscribe(GameEvent.Type.EnemyKilled, OnEnemyKilled);
    }

    private void OnEnemyKilled(object data) => PlaySound("enemy_death");
}
```

**Why queued events?**

- Timing control: Process when convenient, spread load across frames
- Order independence: Senders don't wait for receivers
- Prevents cascading: An event handler can safely queue more events

---

# Data-Driven Patterns

## ScriptableObject Pattern

Use ScriptableObjects for data containers that exist outside of scenes.

**Use Cases:** Game configuration, item databases, shared runtime data, event channels

```csharp
// Data container
[CreateAssetMenu(fileName = "WeaponData", menuName = "Game/Weapon Data")]
public class WeaponData : ScriptableObject
{
    public string weaponName;
    public int damage;
    public float attackSpeed;
    public Sprite icon;
    public AudioClip attackSound;
}

// Runtime variable (shared state without singletons)
[CreateAssetMenu(fileName = "IntVariable", menuName = "Variables/Int")]
public class IntVariable : ScriptableObject
{
    public int value;
    
    public void Add(int amount) => value += amount;
    public void Reset() => value = 0;
}

// Event channel (decoupled events)
[CreateAssetMenu(fileName = "VoidEvent", menuName = "Events/Void Event")]
public class VoidEventChannel : ScriptableObject
{
    public event Action OnEventRaised;
    
    public void RaiseEvent() => OnEventRaised?.Invoke();
}

// Usage
public class Weapon : MonoBehaviour
{
    [SerializeField] private WeaponData _data;
    
    public void Attack()
    {
        // Use _data.damage, _data.attackSound, etc.
    }
}

public class ScoreManager : MonoBehaviour
{
    [SerializeField] private IntVariable _score;
    [SerializeField] private VoidEventChannel _onEnemyKilled;

    private void OnEnable() => _onEnemyKilled.OnEventRaised += AddScore;
    private void OnDisable() => _onEnemyKilled.OnEventRaised -= AddScore;
    
    private void AddScore() => _score.Add(100);
}
```

---

## Type Object Pattern

Define "types" as data instead of code, allowing runtime flexibility.

**Use Cases:** Breed/species systems, item types, spell schools

```csharp
[CreateAssetMenu(menuName = "Game/Monster Breed")]
public class MonsterBreed : ScriptableObject
{
    public string breedName;
    public int baseHealth;
    public int baseAttack;
    public float moveSpeed;
    public string[] abilities;
    public GameObject prefab;
}

public class Monster : MonoBehaviour
{
    [SerializeField] private MonsterBreed _breed;
    
    private int _currentHealth;
    
    private void Awake()
    {
        _currentHealth = _breed.baseHealth;
    }
    
    public int Attack() => _breed.baseAttack;
    public float Speed => _breed.moveSpeed;
}

// Create 100 monster types by making 100 ScriptableObjects
// No code changes needed, designers can do it
```

---

# Optimization Patterns

## Dirty Flag Pattern

Track when data has changed to avoid unnecessary expensive operations.

**Use Cases:** Mesh rebuilding, pathfinding recalculation, UI updates, transform hierarchies

```csharp
public class InventoryUI : MonoBehaviour
{
    private bool _isDirty;
    private List<Item> _items = new();

    public void AddItem(Item item)
    {
        _items.Add(item);
        _isDirty = true;
    }

    public void RemoveItem(Item item)
    {
        _items.Remove(item);
        _isDirty = true;
    }

    private void LateUpdate()
    {
        if (_isDirty)
        {
            RebuildUI();
            _isDirty = false;
        }
    }

    private void RebuildUI()
    {
        // Expensive UI rebuild operation
    }
}

// More complex: Transform hierarchy with world matrix caching
public class TransformNode
{
    private Matrix4x4 _localMatrix;
    private Matrix4x4 _worldMatrix;
    private bool _isDirty = true;
    
    private TransformNode _parent;
    private List<TransformNode> _children = new();

    public Matrix4x4 WorldMatrix
    {
        get
        {
            if (_isDirty)
            {
                _worldMatrix = _parent != null
                    ? _parent.WorldMatrix * _localMatrix
                    : _localMatrix;
                _isDirty = false;
            }
            return _worldMatrix;
        }
    }

    public void SetLocalPosition(Vector3 position)
    {
        _localMatrix = Matrix4x4.Translate(position);
        SetDirty();
    }

    private void SetDirty()
    {
        _isDirty = true;
        foreach (var child in _children)
        {
            child.SetDirty();  // Propagate to children
        }
    }
}
```

---

## Data Locality Pattern

Organize data to maximize CPU cache utilization. Keep data that's accessed together stored together.

**Use Cases:** Entity systems, particle systems, any performance-critical batch processing

```csharp
// BAD: Objects scattered in memory, cache misses everywhere
public class EntityBad
{
    public Vector3 Position;
    public Vector3 Velocity;
    public bool IsActive;
    // Other fields...
}

public class WorldBad
{
    private List<EntityBad> _entities = new();  // References scattered in heap
    
    public void Update(float dt)
    {
        foreach (var entity in _entities)  // Cache miss on every entity
        {
            if (entity.IsActive)
            {
                entity.Position += entity.Velocity * dt;
            }
        }
    }
}

// GOOD: Data-oriented design, contiguous arrays
public class WorldGood
{
    // Parallel arrays - data accessed together is stored together
    private Vector3[] _positions;
    private Vector3[] _velocities;
    private bool[] _isActive;
    private int _count;
    private int _capacity;

    public WorldGood(int capacity)
    {
        _capacity = capacity;
        _positions = new Vector3[capacity];
        _velocities = new Vector3[capacity];
        _isActive = new bool[capacity];
    }

    public int AddEntity(Vector3 position, Vector3 velocity)
    {
        if (_count >= _capacity) return -1;
        
        int index = _count++;
        _positions[index] = position;
        _velocities[index] = velocity;
        _isActive[index] = true;
        return index;
    }

    public void Update(float dt)
    {
        // Contiguous memory access = cache friendly
        for (int i = 0; i < _count; i++)
        {
            if (_isActive[i])
            {
                _positions[i] += _velocities[i] * dt;
            }
        }
    }

    // Even better: Process only active entities (hot/cold separation)
    public void UpdateOptimized(float dt)
    {
        // Process positions and velocities in separate passes
        // Modern CPUs can prefetch and pipeline better this way
        for (int i = 0; i < _count; i++)
        {
            _positions[i] += _velocities[i] * dt;
        }
    }
}
```

**When to use:** Only when you have thousands of entities and profiling shows cache misses are a problem. Don't prematurely optimize.

---

## Spatial Partition Pattern

Organize objects by position for faster lookups. Avoid O(n²) collision checks.

**Use Cases:** Broad-phase collision, finding nearby objects, AI awareness

```csharp
public class SpatialGrid<T> where T : MonoBehaviour
{
    private readonly Dictionary<Vector2Int, List<T>> _cells = new();
    private readonly float _cellSize;

    public SpatialGrid(float cellSize)
    {
        _cellSize = cellSize;
    }

    private Vector2Int GetCell(Vector3 position)
    {
        return new Vector2Int(
            Mathf.FloorToInt(position.x / _cellSize),
            Mathf.FloorToInt(position.z / _cellSize)
        );
    }

    public void Add(T obj)
    {
        var cell = GetCell(obj.transform.position);
        if (!_cells.ContainsKey(cell))
            _cells[cell] = new List<T>();
        _cells[cell].Add(obj);
    }

    public void Remove(T obj)
    {
        var cell = GetCell(obj.transform.position);
        if (_cells.ContainsKey(cell))
            _cells[cell].Remove(obj);
    }

    public void UpdatePosition(T obj, Vector3 oldPosition)
    {
        var oldCell = GetCell(oldPosition);
        var newCell = GetCell(obj.transform.position);
        
        if (oldCell != newCell)
        {
            if (_cells.ContainsKey(oldCell))
                _cells[oldCell].Remove(obj);
            Add(obj);
        }
    }

    public List<T> GetNearby(Vector3 position, int radius = 1)
    {
        var result = new List<T>();
        var center = GetCell(position);
        
        for (int x = -radius; x <= radius; x++)
        {
            for (int y = -radius; y <= radius; y++)
            {
                var cell = new Vector2Int(center.x + x, center.y + y);
                if (_cells.TryGetValue(cell, out var objects))
                    result.AddRange(objects);
            }
        }
        return result;
    }
}

// Usage
public class CollisionManager : MonoBehaviour
{
    private SpatialGrid<Enemy> _enemyGrid;

    private void Awake()
    {
        _enemyGrid = new SpatialGrid<Enemy>(10f);  // 10 unit cells
    }

    public List<Enemy> GetEnemiesNear(Vector3 position, float range)
    {
        // Instead of checking all enemies (O(n)), check only nearby cells (O(1))
        return _enemyGrid.GetNearby(position, Mathf.CeilToInt(range / 10f));
    }
}
```

---

# Quick Reference

| Pattern           | Category     | Primary Use Case                            |
| ----------------- | ------------ | ------------------------------------------- |
| Strategy          | Behavioral   | Swappable algorithms (AI, movement)         |
| State             | Behavioral   | State machines (player, game flow)          |
| Observer          | Behavioral   | Event systems (UI updates, achievements)    |
| Command           | Behavioral   | Input handling, undo/redo, replays          |
| Subclass Sandbox  | Behavioral   | Spell/ability systems with shared utilities |
| Bytecode          | Behavioral   | Data-driven scripting, modding support      |
| Factory           | Creational   | Object instantiation (enemies, items)       |
| Abstract Factory  | Creational   | Themed content families                     |
| Prototype         | Creational   | Cloning objects with variations             |
| Singleton         | Creational   | Global managers (use sparingly)             |
| Object Pool       | Creational   | Performance (bullets, particles)            |
| Builder           | Creational   | Complex object construction                 |
| Component         | Structural   | Entity composition (Unity's core)           |
| Decorator         | Structural   | Runtime modifications (buffs, enchantments) |
| Flyweight         | Structural   | Sharing data (terrain, trees)               |
| Facade            | Structural   | Complex system simplification               |
| Service Locator   | Structural   | Decoupled service access                    |
| Game Loop         | Sequencing   | Core update cycle                           |
| Update Method     | Sequencing   | Per-entity behavior                         |
| Double Buffer     | Sequencing   | Consistent state during updates             |
| Event Queue       | Decoupling   | Deferred, decoupled events                  |
| ScriptableObject  | Data-Driven  | Data containers, events                     |
| Type Object       | Data-Driven  | Designer-configurable types                 |
| Dirty Flag        | Optimization | Avoiding redundant updates                  |
| Data Locality     | Optimization | Cache-friendly data layout                  |
| Spatial Partition | Optimization | Position-based queries                      |

---

## When to Use What

### Behavior & State Problems

**Need swappable algorithms?** → Strategy  
**Need object behavior to change with its state?** → State  
**Need to notify multiple objects of changes?** → Observer  
**Need deferred or decoupled event handling?** → Event Queue  
**Need undo/redo or replay systems?** → Command

### Object Creation Problems

**Need to spawn many objects efficiently?** → Factory + Object Pool  
**Need copies of objects with variations?** → Prototype  
**Need to construct complex objects step by step?** → Builder  
**Need one global instance?** → Singleton _(use sparingly)_

### Structure & Decoupling Problems

**Need to compose entities from reusable parts?** → Component  
**Need to add behavior without subclassing?** → Decorator  
**Need to simplify a complex subsystem?** → Facade  
**Need global service access without tight coupling?** → Service Locator _(prefer DI)_

### Data-Driven Design Problems

**Need designer-configurable data?** → ScriptableObjects  
**Need types defined as data, not code?** → Type Object  
**Need shared utilities for ability/spell systems?** → Subclass Sandbox  
**Need fully scriptable behaviors or modding support?** → Bytecode

### Performance & Optimization Problems

**Need memory efficiency with many similar objects?** → Flyweight  
**Need to avoid redundant expensive updates?** → Dirty Flag  
**Need cache-friendly data layout?** → Data Locality  
**Need fast position-based queries?** → Spatial Partition

### Sequencing & Timing Problems

**Need consistent state during simultaneous updates?** → Double Buffer  
**Need to understand the core update cycle?** → Game Loop  
**Need per-entity frame behavior?** → Update Method