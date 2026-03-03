# Unity Script Naming Patterns

Common suffix and prefix conventions for C# scripts in Unity projects.

---

**Suffix** — appended to the concept name:​
- `SpellData`, `ItemData`, `EnemyData`
- `SpellConfig`, `PlayerConfig`
- `SpellDefinition`

**Prefix** — prepended identifier for the asset type:
- `SO_SpellData`, `SO_PlayerConfig`
- `D_Spell`, `D_Item` (D for Data)

## Common Script Suffixes

| Suffix | Typical Role | Example |
|---|---|---|
| `Manager` | Singleton/global that owns a system's state and lifecycle | `GameManager`, `AudioManager`, `UIManager` |
| `Controller` | Drives a single entity's behavior (often via input or AI) | `PlayerController`, `CameraController`, `EnemyController` |
| `Handler` | Responds to events or callbacks | `InputHandler`, `DamageHandler`, `CollisionHandler` |
| `System` | Processes entities/data each frame (ECS-style or broad systems) | `MovementSystem`, `SpawnSystem`, `InventorySystem` |
| `Data` | Plain data container (often a class/struct, not MonoBehaviour) | `PlayerData`, `WeaponData`, `LevelData` |
| `Config` | Configuration values, often a [[ScriptableObjects\|ScriptableObject]] | `DifficultyConfig`, `AudioConfig`, `GameConfig` |
| `SO` | ScriptableObject asset | `WeaponSO`, `EnemySO`, `AbilitySO` |
| `Settings` | Editable parameters (similar to Config) | `GraphicsSettings`, `InputSettings` |
| `UI` | UI-specific logic | `HealthBarUI`, `InventoryUI`, `DialogueUI` |
| `View` | Visual representation (in MVC/MVP patterns) | `PlayerView`, `ShopView`, `CardView` |
| `Model` | Data model (in MVC/MVP patterns) | `PlayerModel`, `InventoryModel` |
| `Presenter` | Mediates between Model and View | `ShopPresenter`, `InventoryPresenter` |
| `Factory` | Creates and configures instances | `EnemyFactory`, `ProjectileFactory`, `UIFactory` |
| `Spawner` | Instantiates objects at runtime (often positional) | `EnemySpawner`, `ItemSpawner`, `WaveSpawner` |
| `Pool` / `ObjectPool` | Manages reusable object instances | `BulletPool`, `VFXPool`, `EnemyPool` |
| `Trigger` | Responds to physics trigger volumes | `DamageTrigger`, `CheckpointTrigger`, `DialogueTrigger` |
| `Detector` | Senses environment (raycasts, overlap checks) | `GroundDetector`, `LineOfSightDetector` |
| `State` | Single state in a state machine | `IdleState`, `AttackState`, `DeadState` |
| `StateMachine` | Manages state transitions | `PlayerStateMachine`, `AIStateMachine` |
| `Component` | Generic reusable component | `HealthComponent`, `DamageComponent` |
| `Behaviour` / `Behavior` | MonoBehaviour with specific behavior | `PatrolBehaviour`, `FleeBehaviour` |
| `Helper` / `Utility` / `Utils` | Static utility methods | `MathHelper`, `VectorUtils`, `StringUtility` |
| `Extension` / `Extensions` | C# extension methods | `VectorExtensions`, `TransformExtensions` |
| `Editor` | Custom editor/inspector script (in Editor folder) | `WeaponEditor`, `LevelEditor`, `CustomInspectorEditor` |
| `Drawer` | Custom PropertyDrawer | `RangeDrawer`, `TagDrawer` |
| `Window` | Custom EditorWindow | `LevelEditorWindow`, `DebugWindow` |
| `Attribute` | Custom C# attribute | `ReadOnlyAttribute`, `RequireInterfaceAttribute` |
| `Info` | Lightweight data record | `SpawnInfo`, `HitInfo`, `WaveInfo` |
| `Args` / `EventArgs` | Event argument data | `DamageEventArgs`, `InteractionArgs` |
| `Provider` | Supplies data or services to consumers | `TimeProvider`, `InputProvider` |
| `Service` | Encapsulated subsystem accessed by interface | `AudioService`, `SaveService`, `AnalyticsService` |
| `Resolver` / `Locator` | Service locator pattern | `ServiceLocator`, `DependencyResolver` |

---

## Common Script Prefixes

| Prefix | Usage | Example |
|---|---|---|
| `Base` | Abstract base class | `BaseEnemy`, `BaseWeapon`, `BaseUI` |
| `Abstract` | Abstract base class (alternative) | `AbstractState`, `AbstractFactory` |
| `I` | Interface (C# convention) | `IDamageable`, `IInteractable`, `ISaveable` |
| `SO_` | ScriptableObject (alternative to suffix) | `SO_Weapon`, `SO_DialogueLine` |

---

## Naming Conventions

| Element | Convention | Example |
|---|---|---|
| Class name | PascalCase | `PlayerController` |
| Public field | camelCase or PascalCase (varies) | `moveSpeed` or `MoveSpeed` |
| Private field | `_camelCase` (underscore prefix) | `_currentHealth` |
| Serialized private field | `_camelCase` with `[SerializeField]` | `[SerializeField] private float _speed;` |
| Property | PascalCase | `public float Health { get; set; }` |
| Method | PascalCase | `TakeDamage()`, `Initialize()` |
| Constant | PascalCase or UPPER_SNAKE | `MaxHealth` or `MAX_HEALTH` |
| Enum | PascalCase (type and values) | `enum WeaponType { Sword, Bow, Staff }` |
| Event | PascalCase with `On` prefix | `OnDeath`, `OnDamageReceived` |
| Action/Callback field | PascalCase | `Action<int> OnHealthChanged` |
| File name | Must match class name exactly | `PlayerController.cs` |

---

## Sources
- [Unity Manual: Creating and Using Scripts](https://docs.unity3d.com/Manual/CreatingAndUsingScripts.html)
- [Microsoft C# Naming Guidelines](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/coding-conventions)

## Related topics
- [[Unity Project Structure Reference]]
- [[Csharp Fundamentals Reference]]
- [[ScriptableObjects]]
- [[MonoBehaviour Reference]]
