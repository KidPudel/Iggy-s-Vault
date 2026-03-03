# Unity Project Structure Reference

Folder layout, special folders, assembly definitions, and Godot project structure mapping.

---

## Common Folder Structure

```
Assets/
├── _Project/              # or _Game/ — project-specific root
│   ├── Scenes/
│   ├── Scripts/
│   │   ├── Player/
│   │   ├── Enemies/
│   │   ├── UI/
│   │   ├── Systems/
│   │   ├── Data/
│   │   └── Utilities/
│   ├── Prefabs/
│   ├── Materials/
│   ├── Textures/
│   ├── Audio/
│   │   ├── Music/
│   │   └── SFX/
│   ├── Animations/
│   ├── Shaders/
│   ├── ScriptableObjects/
│   ├── UI/
│   ├── Fonts/
│   ├── VFX/
│   └── Resources/         # only for assets loaded via Resources.Load()
├── Plugins/                # native plugins
├── Editor/                 # editor-only scripts
│   └── Tools/
├── StreamingAssets/        # raw files copied to build
└── Third Party/            # or Vendors/ — imported packages/assets
```

The `_` prefix on `_Project` sorts it to the top in the Unity Project window.

---

## Special Folders

| Folder | Compile Order | In Build? | Behavior |
|---|---|---|---|
| `Editor/` | After runtime | No | Editor-only scripts. Can be nested anywhere. |
| `Editor Default Resources/` | — | No | Editor assets loaded via `EditorGUIUtility.Load()` |
| `Resources/` | Normal | Yes (all contents) | Runtime-loadable via `Resources.Load()` |
| `StreamingAssets/` | — | Yes (verbatim) | Not processed. Accessed via `Application.streamingAssetsPath` |
| `Plugins/` | Before other scripts | Yes | Native DLLs. Compiled in first pass. |
| `Standard Assets/` | Before other scripts | Yes | Legacy. Compiled in first pass. |
| `Gizmos/` | — | No | Custom gizmo textures for Scene view |
| `Hidden Assets/` | — | No | Folders starting with `.` or ending with `~` are ignored |

> [!info]
> Files named with a leading `.` or ending in `~`, and folders named `cvs` or `.svn`, are ignored by Unity's import pipeline.

---

## Assembly Definitions (.asmdef)

Assembly definition files split the project into separate compilation units.

### File: `*.asmdef`

```json
{
    "name": "MyGame.Player",
    "rootNamespace": "MyGame.Player",
    "references": [
        "MyGame.Core",
        "MyGame.Utils"
    ],
    "includePlatforms": [],
    "excludePlatforms": [],
    "allowUnsafeCode": false,
    "overrideReferences": false,
    "precompiledReferences": [],
    "autoReferenced": true,
    "defineConstraints": [],
    "versionDefines": [],
    "noEngineReferences": false
}
```

### .asmdef Properties

| Property | Type | Description |
|---|---|---|
| `name` | string | Assembly name (must be unique) |
| `rootNamespace` | string | Default namespace for new scripts |
| `references` | string[] | Other assembly definitions this depends on |
| `includePlatforms` | string[] | Only compile for these platforms (empty = all) |
| `excludePlatforms` | string[] | Exclude these platforms |
| `allowUnsafeCode` | bool | Allow `unsafe` C# code |
| `overrideReferences` | bool | Use explicit precompiled references |
| `precompiledReferences` | string[] | Specific DLL references when overrideReferences is true |
| `autoReferenced` | bool | Automatically referenced by predefined assemblies |
| `defineConstraints` | string[] | Only compile when these defines are set |
| `noEngineReferences` | bool | Exclude UnityEngine references |

### Assembly Reference File: `*.asmref`

Points to an existing `.asmdef` to include scripts from a different folder in that assembly:

```json
{
    "reference": "MyGame.Core"
}
```

### Default Assemblies (without .asmdef)

| Assembly | Contents |
|---|---|
| `Assembly-CSharp-firstpass` | Scripts in `Plugins/` and `Standard Assets/` |
| `Assembly-CSharp-Editor-firstpass` | Editor scripts in `Plugins/Editor/` and `Standard Assets/Editor/` |
| `Assembly-CSharp` | All other runtime scripts |
| `Assembly-CSharp-Editor` | All other Editor scripts |

> [!info]
> Scripts in `Assembly-CSharp` (no .asmdef) can reference any assembly. Scripts in a custom .asmdef can only reference assemblies explicitly listed in `references`.

---

## Typical .asmdef Layout

```
Assets/_Project/
├── Scripts/
│   ├── Core/
│   │   └── MyGame.Core.asmdef
│   ├── Player/
│   │   └── MyGame.Player.asmdef      → references: MyGame.Core
│   ├── Enemies/
│   │   └── MyGame.Enemies.asmdef     → references: MyGame.Core
│   ├── UI/
│   │   └── MyGame.UI.asmdef          → references: MyGame.Core
│   └── Editor/
│       └── MyGame.Editor.asmdef      → references: MyGame.Core (Editor-only platforms)
└── Tests/
    ├── EditMode/
    │   └── MyGame.Tests.EditMode.asmdef
    └── PlayMode/
        └── MyGame.Tests.PlayMode.asmdef
```

---

## Godot Project Structure Mapping

| Unity | Godot |
|---|---|
| `Assets/` | `res://` (project root) |
| `Assets/Scenes/` | Scenes (`.tscn`) anywhere, often in `res://scenes/` |
| `Assets/Scripts/` | Scripts (`.gd`) often live next to their scene or in `res://scripts/` |
| `Assets/Prefabs/` | Packed scenes (`.tscn`) reused via instantiation |
| `Assets/Resources/` | Any `res://` path (all resources loadable by default) |
| `Assets/StreamingAssets/` | `user://` for runtime files, or export filters |
| `Assets/Editor/` | `@tool` scripts, `addons/` folder |
| `Assets/Plugins/` | `addons/` folder |
| `ProjectSettings/` | `project.godot`, `export_presets.cfg` |
| `Library/` | `.godot/` (cached imports) |
| `Packages/` (UPM) | AssetLib or git submodules in `addons/` |
| `.asmdef` files | No equivalent (single compilation unit per language) |

### Typical Godot Layout

```
res://
├── project.godot
├── scenes/
│   ├── levels/
│   └── ui/
├── scripts/
│   ├── player/
│   ├── enemies/
│   └── autoload/
├── assets/
│   ├── textures/
│   ├── audio/
│   └── fonts/
├── shaders/
├── addons/
└── export_presets.cfg
```

---

## Sources
- [Unity Manual: Special Folder Names](https://docs.unity3d.com/Manual/SpecialFolders.html)
- [Unity Manual: Assembly Definitions](https://docs.unity3d.com/Manual/ScriptCompilationAssemblyDefinitionFiles.html)
- [Unity Manual: Prefabs](https://docs.unity3d.com/Manual/Prefabs.html)
- [Unity Manual: ScriptableObject](https://docs.unity3d.com/Manual/class-ScriptableObject.html)

## Related topics
- [[godot project structure recommendation]]
- [[Unity assets reference]]
- [[ScriptableObjects]]
