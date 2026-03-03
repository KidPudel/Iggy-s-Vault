# Unity Assets Reference

Asset types, file extensions, import settings, meta files, and special project folders.

---

## Asset Types

| Asset Type | Common Extensions | Godot Equivalent |
|---|---|---|
| Textures | `.png`, `.jpg`, `.psd`, `.tga`, `.exr`, `.hdr`, `.tiff`, `.bmp`, `.gif` | Same formats, imported as `Image`/`CompressedTexture2D` |
| 3D Models | `.fbx`, `.obj`, `.dae`, `.blend`, `.3ds`, `.gltf`, `.glb` | `.glb`/`.gltf` preferred, `.blend` with plugin |
| Audio | `.wav`, `.mp3`, `.ogg`, `.aif`, `.flac` | `.wav`, `.ogg`, `.mp3` |
| Video | `.mp4`, `.mov`, `.webm`, `.avi` | `.ogv`, `.webm` |
| Fonts | `.ttf`, `.otf` | `.ttf`, `.otf` |
| Materials | `.mat` (Unity-specific) | `.tres` / `.material` |
| Shaders | `.shader`, `.shadergraph`, `.hlsl`, `.cginc`, `.compute` | `.gdshader` |
| Prefabs | `.prefab` (Unity-specific) | `.tscn` (packed scene) |
| Scenes | `.unity` (Unity-specific) | `.tscn` |
| Animations | `.anim`, `.controller` (Unity-specific) | `.tres` / `.res` |
| ScriptableObjects | `.asset` (Unity-specific) | `Resource` saved as `.tres` |
| Text/Data | `.json`, `.xml`, `.csv`, `.txt`, `.yaml`, `.bytes` | Same formats |
| Scripts | `.cs` | `.gd`, `.cs` |
| Assembly Definitions | `.asmdef`, `.asmref` | No equivalent |
| Sprite Atlases | `.spriteatlas` | Atlas in `TextureAtlas` |

---

## Meta Files (.meta)

Every asset and folder in Unity gets a `.meta` file containing:
- **GUID** — unique persistent identifier for the asset
- **Import settings** — texture compression, model scale, audio format, etc.

> [!warning]
> Never delete `.meta` files manually. Deleting a `.meta` file breaks all references to that asset across the project. Always commit `.meta` files to version control.

Godot equivalent: `.import` files (auto-generated, also should be committed to VCS).

---

## Common Import Settings by Type

### Textures

| Setting | Options |
|---|---|
| Texture Type | Default, Normal Map, Editor GUI, Sprite, Cursor, Cookie, Lightmap, Single Channel |
| sRGB | On/Off (Off for normal maps, masks) |
| Max Size | 32–16384 |
| Compression | None, Low Quality, Normal Quality, High Quality |
| Filter Mode | Point, Bilinear, Trilinear |
| Wrap Mode | Repeat, Clamp, Mirror, MirrorOnce |
| Generate Mip Maps | On/Off |
| Read/Write | On/Off (Off = less memory, On = CPU access) |
| Sprite Mode | Single, Multiple, Polygon |

### 3D Models

| Setting | Options |
|---|---|
| Scale Factor | Default 1 (FBX default 0.01) |
| Import BlendShapes | On/Off |
| Import Visibility | On/Off |
| Import Cameras | On/Off |
| Import Lights | On/Off |
| Mesh Compression | Off, Low, Medium, High |
| Read/Write | On/Off |
| Generate Colliders | On/Off |
| Animation Import | On/Off |
| Rig → Animation Type | None, Legacy, Generic, Humanoid |
| Materials → Import | On/Off |

### Audio

| Setting | Options |
|---|---|
| Force To Mono | On/Off |
| Load In Background | On/Off |
| Load Type | Decompress On Load, Compressed In Memory, Streaming |
| Compression Format | PCM, ADPCM, Vorbis |
| Quality | 1–100 (Vorbis only) |
| Sample Rate Setting | Preserve, Optimize, Override |

> [!info]
> Short SFX: Decompress On Load. Music/long audio: Streaming. Medium clips: Compressed In Memory.

---

## Special Folders

| Folder | Behavior |
|---|---|
| `Assets/` | Root project folder. All assets live here. |
| `Assets/Editor/` | Scripts here only run in the Unity Editor, excluded from builds. Any subfolder named `Editor` works. |
| `Assets/Editor Default Resources/` | Assets loadable via `EditorGUIUtility.Load()`. Editor-only. |
| `Assets/Gizmos/` | Custom gizmo icons for the Scene view. |
| `Assets/Plugins/` | Native plugin DLLs (`.dll`, `.so`, `.dylib`, `.a`). Compiled first. |
| `Assets/Resources/` | Assets loadable at runtime via `Resources.Load("path")`. All contents included in build regardless of scene references. |
| `Assets/StreamingAssets/` | Files copied verbatim into build (not processed). Access via `Application.streamingAssetsPath`. |
| `Assets/Standard Assets/` | Compiled before other scripts (legacy). |
| `Packages/` | Unity Package Manager packages (outside `Assets/`). |
| `ProjectSettings/` | Editor and project configuration files. |
| `Library/` | Local cache. Do not commit to VCS. Regenerated from `Assets/` + `Packages/`. |
| `Logs/` | Editor and compiler logs. |
| `Temp/` | Temporary build files. |
| `UserSettings/` | Per-user editor preferences. |

### Godot Special Folder Mapping

| Unity | Godot |
|---|---|
| `Assets/` | `res://` (project root) |
| `Assets/Resources/` | No direct equivalent; all `res://` resources loadable via `load()`/`preload()` |
| `Assets/StreamingAssets/` | `res://` files with `.import` skip, or `user://` |
| `Assets/Editor/` | `addons/` (for plugins), `@tool` scripts |
| `Library/` | `.godot/` (imported cache) |

---

## Resources.Load() Path Rules

```csharp
// Path is relative to any Resources folder, no extension
GameObject prefab = Resources.Load<GameObject>("Enemies/Goblin");
Sprite icon = Resources.Load<Sprite>("UI/Icons/health");
TextAsset data = Resources.Load<TextAsset>("Data/config");

// Load all of type
Object[] all = Resources.LoadAll<Sprite>("UI/Icons");
```

- Path separators: forward slash `/` on all platforms
- No file extension in the path
- Multiple `Resources/` folders can exist (merged at build time)
- Everything in `Resources/` is included in the build, even if unused in scenes

---

## Addressable Assets (Summary)

| Concept | Description |
|---|---|
| Address | String key assigned to an asset |
| Label | Tag for grouping assets |
| AssetReference | Serializable reference to an addressable |
| Loading | `Addressables.LoadAssetAsync<T>(key)` |
| Instantiation | `Addressables.InstantiateAsync(key)` |
| Release | `Addressables.Release(handle)` |

---

## Sources
- [Unity Manual: Asset Workflow](https://docs.unity3d.com/Manual/AssetWorkflow.html)
- [Unity Manual: Importing Assets](https://docs.unity3d.com/Manual/ImportingAssets.html)
- [Unity Manual: Special Folder Names](https://docs.unity3d.com/Manual/SpecialFolders.html)
- [Unity Manual: Assets and Resources](https://docs.unity3d.com/Manual/AssetTypes.html)

## Related topics
- [[Prefabs]]
- [[ScriptableObjects]]
- [[Serialization and Inspector in Unity]]
- [[Unity Index Book of References]]
