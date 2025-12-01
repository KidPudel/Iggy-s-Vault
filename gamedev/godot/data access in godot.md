# Godot 4 Data Access Reference

## Path Prefixes

|Prefix|Use|Writable|After Export|
|---|---|---|---|
|`res://`|Project assets|Editor only|Read-only (in PCK)|
|`user://`|Saves, configs|Always|Always|

```gdscript
OS.get_user_data_dir()                    # Absolute path to user://
ProjectSettings.globalize_path("user://") # Convert to absolute
```

---

## Loading Resources — When to Use What

|Method|When to Use|Blocks?|
|---|---|---|
|`preload()`|Small, always-needed assets. Path must be literal string.|At script load|
|`load()`|Dynamic paths, conditional loading|Yes|
|`ResourceLoader.load()`|Need cache control or type hints|Yes|
|`ResourceLoader.load_threaded_*()`|Large files, avoid stutters|No|

```gdscript
# preload — compile-time, path must be literal
const BULLET = preload("res://bullet.tscn")

# load — runtime, blocks
var scene = load("res://levels/" + level_name + ".tscn")

# ResourceLoader — same as load() but with options
var tex = ResourceLoader.load("res://img.png", "Texture2D", ResourceLoader.CACHE_MODE_IGNORE)

# Threaded — non-blocking
ResourceLoader.load_threaded_request("res://big.tscn")
# Later in _process:
if ResourceLoader.load_threaded_get_status("res://big.tscn") == ResourceLoader.THREAD_LOAD_LOADED:
    var scene = ResourceLoader.load_threaded_get("res://big.tscn")
```

**Cache modes:** `CACHE_MODE_REUSE` (default), `CACHE_MODE_REPLACE`, `CACHE_MODE_IGNORE`

---

## File I/O — `FileAccess`

**When:** Raw files (text, binary, JSON), save data, configs — anything that's not a Godot resource.

```gdscript
# Write
var f = FileAccess.open("user://save.dat", FileAccess.WRITE)
f.store_var({"level": 5, "pos": Vector2(10, 20)})  # Any Variant

# Read
var f = FileAccess.open("user://save.dat", FileAccess.READ)
var data = f.get_var()  # Deserializes back

# Check existence
FileAccess.file_exists("user://save.dat")

# Error handling
if FileAccess.open("user://x.txt", FileAccess.READ) == null:
    print(FileAccess.get_open_error())
```

**Modes:** `READ`, `WRITE`, `READ_WRITE`, `WRITE_READ`

---

## Directory Listing — `DirAccess` vs `ResourceLoader.list_directory()`

### The Problem

After export, `res://` files get remapped (`.png` → `.png.import`). `DirAccess` sees the remapped names; `ResourceLoader.list_directory()` sees original names.

|Method|Use Case|Sees After Export|
|---|---|---|
|`DirAccess`|`user://`, filesystem, modding|Actual files (`.import`, `.remap`)|
|`ResourceLoader.list_directory()`|`res://` resources|Original names (`image.png`)|

```gdscript
# For res:// after export — use this
var files = ResourceLoader.list_directory("res://sprites/")
# Returns: ["player.png", "enemy.png"] (usable paths)

# For user:// or actual filesystem
var dir = DirAccess.open("user://saves/")
var files = dir.get_files()  # or get_directories()
```

### DirAccess Workaround (if needed for res://)

```gdscript
# Strip .import/.remap suffix to get loadable path
var filename = "sprite.png.import".trim_suffix(".import").trim_suffix(".remap")
var res = load("res://sprites/" + filename)
```

### DirAccess Operations

```gdscript
DirAccess.make_dir_recursive_absolute("user://saves/slot1")
DirAccess.copy_absolute("user://a.txt", "user://b.txt")
DirAccess.remove_absolute("user://old.txt")
DirAccess.dir_exists_absolute("user://saves")
```

---

## Custom Resources — Best for Typed Save Data

```gdscript
# Define (player_data.gd)
class_name PlayerData extends Resource
@export var level := 1
@export var inventory: Array[String] = []

# Save
var data = PlayerData.new()
data.level = 5
ResourceSaver.save(data, "user://save.tres")

# Load
var data = load("user://save.tres") as PlayerData
```

---

## Quick Reference

|Task|Use|
|---|---|
|Load scene/texture/audio|`load()` / `preload()`|
|Load without blocking|`ResourceLoader.load_threaded_*()`|
|Read/write raw files|`FileAccess`|
|Save game data|`ResourceSaver` + custom Resource, or `FileAccess.store_var()`|
|List `res://` files (export-safe)|`ResourceLoader.list_directory()`|
|List `user://` files|`DirAccess.get_files()`|
|Create/delete/copy files|`DirAccess`|

---

## Export Pitfalls

| Issue                                 | Fix                                                 |
| ------------------------------------- | --------------------------------------------------- |
| Can't read `res://` file after export | Use `ResourceLoader`, not `FileAccess`              |
| `DirAccess` shows `.import` files     | Use `ResourceLoader.list_directory()` for `res://`  |
| Need editable files after export      | Store in `user://`                                  |
| External mod support                  | Load from `OS.get_executable_path().get_base_dir()` |