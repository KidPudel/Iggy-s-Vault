# Project Structure in Unity

> [!quote] The Core Insight Organize by feature, not by file type. Delete a folder, delete a feature.

---

## The Anti-Pattern

```
Assets/
├── Scripts/
│   ├── PlayerController.cs
│   ├── EnemyAI.cs
│   ├── HealthSystem.cs
│   └── ... (200 more files)
├── Prefabs/
│   ├── Player.prefab
│   ├── Enemy.prefab
│   └── ...
├── Sprites/
│   ├── player_idle.png
│   ├── enemy_walk.png
│   └── ...
└── Audio/
    └── ...
```

**Problem:** Player-related files scattered across 4+ folders. Finding everything for one feature requires searching everywhere.

---

## Feature-Based Structure

```
Assets/
├── _Project/                    # Your game (underscore = top of list)
│   ├── Common/                  # Shared utilities
│   ├── Systems/                 # Pure C# systems
│   ├── Managers/                # MonoBehaviour managers
│   │
│   ├── Player/                  # Everything player-related
│   │   ├── Player.prefab
│   │   ├── PlayerController.cs
│   │   ├── PlayerInput.cs
│   │   ├── Art/
│   │   │   ├── player_idle.png
│   │   │   └── player_run.png
│   │   └── Audio/
│   │       └── footstep.wav
│   │
│   ├── Enemies/
│   │   ├── Enemy.prefab
│   │   ├── EnemyAI.cs
│   │   ├── Goblin/
│   │   │   ├── Goblin.prefab
│   │   │   └── goblin.png
│   │   └── Orc/
│   │
│   ├── UI/
│   ├── Levels/
│   └── Data/                    # ScriptableObject assets
│
├── Plugins/                     # Third-party code
└── Resources/                   # ONLY runtime-loaded assets
```

**Principle:** Delete `Player/` folder → player feature is gone. No orphans elsewhere.

---

## Top-Level Folders

|Folder|Purpose|
|---|---|
|`_Project/`|All your game code and content|
|`Plugins/`|Third-party packages, libraries|
|`Resources/`|Assets loaded by path at runtime (use sparingly)|
|`StreamingAssets/`|Files that stay as-is (JSON, videos)|
|`Editor/`|Editor-only scripts (anywhere named Editor)|

**Underscore prefix** keeps `_Project` at top of file list.

---

## Inside _Project

|Folder|Contents|
|---|---|
|`Common/`|Shared utilities, extensions, base classes|
|`Systems/`|Pure C# logic (see [[Layered Architecture in Unity]])|
|`Managers/`|MonoBehaviour coordinators|
|`Data/`|ScriptableObject assets|
|`UI/`|All UI prefabs, scripts, sprites|
|`Levels/`|Scene files and level-specific content|
|`[Feature]/`|Feature folders (Player, Enemies, etc.)|

---

## Feature Folder Contents

```
Player/
├── Player.prefab              # Main prefab
├── PlayerController.cs        # Core script
├── PlayerInput.cs             # Input handling
├── PlayerData.asset           # ScriptableObject config
├── Art/
│   ├── player_idle.png
│   ├── player_run.png
│   └── Materials/
│       └── player_mat.mat
├── Audio/
│   ├── jump.wav
│   └── hurt.wav
└── Animations/
    ├── PlayerAnimator.controller
    └── player_idle.anim
```

Keep everything for one feature **together**.

---

## Naming Conventions

|Item|Convention|Example|
|---|---|---|
|Folders|PascalCase|`PlayerCharacter/`|
|C# scripts|PascalCase|`PlayerController.cs`|
|Prefabs|PascalCase|`Enemy.prefab`|
|ScriptableObjects|PascalCase|`SwordData.asset`|
|Scenes|PascalCase|`Level01_Forest.unity`|
|Sprites/Textures|snake_case|`player_idle.png`|
|Materials|PascalCase or snake|`PlayerMaterial.mat`|
|Private fields|_camelCase|`private int _health;`|

---

## Assembly Definitions

For larger projects, use `.asmdef` files to:

- Speed up compilation (only recompile what changed)
- **Enforce architectural boundaries** (prevent bad dependencies)
- Enable unit testing

### Basic Setup

```
Assets/
├── _Project/
│   ├── _Project.asmdef          # Main game assembly
│   ├── Core/
│   │   └── Core.asmdef          # Core systems (no gameplay deps)
│   ├── Gameplay/
│   │   └── Gameplay.asmdef      # Gameplay code (depends on Core)
│   └── Tests/
│       └── Tests.asmdef         # Unit tests
```

### Architectural Enforcement

Assembly definitions physically prevent bad dependencies. If Core tries to reference Gameplay, **the code won't compile**.

```
Core.asmdef
├── References: (none or Unity only)
└── Contains: Audio, Save System, Input, Services

Gameplay.asmdef
├── References: Core
└── Contains: Player, Enemies, Weapons, Levels
```

**The rule:** Core can NEVER see Gameplay. Gameplay CAN see Core.

This prevents "upward coupling" — a generic audio service can never accidentally depend on a specific enemy type.

```csharp
// In Core assembly — this WON'T COMPILE
public class AudioService
{
    public void PlayEnemySound(Enemy enemy)  // ERROR: Enemy is in Gameplay
    {
    }
}

// Correct: Core doesn't know about Enemy
public class AudioService
{
    public void PlaySound(AudioClip clip) { }  // Works — no Gameplay dependency
}
```

### Circular Dependency Prevention

If Assembly A references B, and B tries to reference A → **compile error**.

This forces clean, linear dependency trees. No spaghetti.

### When to Add

|Situation|Recommendation|
|---|---|
|Solo prototype|Skip — overhead not worth it|
|Recompilation annoying (5+ seconds)|Add for speed|
|Team project|Add for architectural enforcement|
|Building reusable packages|Required|

### Creating Assembly Definitions

Right-click folder → Create → Assembly Definition

Configure in Inspector:

- **Name:** Assembly name (e.g., "MyGame.Core")
- **References:** Other assemblies this one can access
- **Platforms:** Which platforms to include (usually all)

---

## Scene Organization

### Hierarchy Structure

```
[MANAGERS]                      # Empty organizer
├── GameManager
├── AudioManager
└── UIManager

[ENVIRONMENT]
├── Ground
├── Walls
└── Props

[ENTITIES]
├── Player
└── Enemies
    ├── Goblin
    └── Orc

[UI]
├── HUD
└── PauseMenu
```

**Brackets `[ ]`** indicate organizational empty GameObjects.

### Scene Files

```
Scenes/
├── Boot.unity              # Entry point
├── Persistent.unity        # Always-loaded managers
├── MainMenu.unity
├── Gameplay.unity          # Core gameplay systems
└── Levels/
    ├── Level01.unity
    └── Level02.unity
```

See [[Scene Management in Unity]] for loading patterns.

---

## What Goes Where

### Scripts

|Type|Location|
|---|---|
|Pure C# systems|`_Project/Systems/`|
|MonoBehaviour managers|`_Project/Managers/`|
|Feature-specific scripts|With the feature|
|Shared utilities|`_Project/Common/`|
|Editor scripts|Any folder named `Editor/`|

### Data

|Type|Location|
|---|---|
|ScriptableObject assets|`_Project/Data/` or with feature|
|JSON/XML config|`StreamingAssets/` or `Data/`|

### Art & Audio

|Type|Location|
|---|---|
|Feature-specific|With the feature|
|Shared/global|`_Project/Art/Shared/`|

---

## Resources Folder — Use Sparingly

Files in `Resources/` can be loaded by path:

```csharp
var prefab = Resources.Load<GameObject>("Prefabs/Enemy");
```

**Problems:**

- All Resources included in build (bloat)
- No compile-time checking
- Can't see what's used

**Prefer:**

- `[SerializeField]` references (compile-time safe)
- Addressables (for large projects)

Only use Resources when you truly need runtime path-based loading.

---

## .gitignore Essentials

```gitignore
# Unity generated
[Ll]ibrary/
[Tt]emp/
[Oo]bj/
[Bb]uild/
[Bb]uilds/
[Ll]ogs/
[Mm]emoryCaptures/

# IDE
.vs/
.idea/
*.csproj
*.sln

# OS
.DS_Store
Thumbs.db

# User-specific
UserSettings/
```

---

## Common Pitfalls

|Pitfall|Problem|Fix|
|---|---|---|
|Everything in Scripts/|Hard to find related files|Organize by feature|
|Huge Resources folder|Build bloat|Use [SerializeField]|
|No assembly definitions|Slow recompilation|Add asmdef files|
|Scenes at root|Clutter|Put in Scenes/ folder|
|Mixed naming conventions|Confusion|Pick one style, stick to it|

---

## Key Takeaways

1. **Feature folders** — group related files together
2. **`_Project/`** — your code, separate from plugins
3. **Avoid Resources/** — prefer direct references
4. **Assembly definitions** — for larger projects
5. **Consistent naming** — PascalCase for most, snake_case for art
6. **Delete folder = delete feature** — no scattered dependencies