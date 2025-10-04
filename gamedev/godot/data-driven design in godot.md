Case: You are picking up an axe from the world (some Node3D, with area and script on picking up logic), obviously we don't want to collect the same object, but rather a **data representation** of the object, that doesn't care about anything, except for data it has itself.
This is a perfect case for [[data-driven design]] to be introduced in Godot.

In Godot to do so, we can simply create a data model, which could be a literal data structure, that acts as a database stored in some basic script/class
```gdscript
class_name Items

const Database = {
	"pickaxe": {
		"name": "Pickaxe"
		"scene": "res://path/to/pickaxe.glb"
	},
	"sword": {
		"name": "Sword"
		"scene": "res://path/to/sword.glb"
	}
}
```

But this approach has several crucial problems:
1. We can easily create a typo in node that uses this like type"swort", which will be a runtime error
2. We can simply move scenes to other places, and the same error will happen
3. Maintaining 100 objects is still hard taking into account 1st and 2nd problems.

We can utilize godot specific feature called [[resource]], which is a data class/structure, which allows for modeling a [[static]] data.
```gdscript
class_name Item extends Resource

@export var name: String
@export var scene: PackedScene # godot will automatically fix the path if we move it somewhere
```

Which then allows as to create specific resources based on our new `Item` type, like `Pickaxe`, `Sword`, etc. and now we can securely reference concrete setup resources.

**If your CardResource instances exist only inside a ColumnResource and nowhere else, there’s no need to save them individually as separate .tres or .tscn files.**


# immutable config vs mutable saving state of progress

static data like the template, definition, during design time is stored in `res://` is immutable, while saving state or just mutable data should be in the `user://`


## **`res://`**
- This is your **project’s resources path**.
- It points to the **read-only bundle of files shipped with your game** (all your .tres, .scn, .gd, textures, sounds, etc.).
- When you run in the editor, res:// maps to your project folder on disk.
    When you export the game, everything in res:// is packed into a .pck or .zip file.
    On a player’s machine, that package is **read-only**.
- If you try to write a save file into res://, it’ll work in the editor (since it’s just your filesystem), but fail or be ignored in a built game — because the player doesn’t have write permission to your packed assets.

**Usage:** store only **static, authored assets**. Config .tres, .tscn, art, sound, scripts. Immutable design data.

---

## **`user://`**
- This is the **per-user writable data folder** Godot provides.
- Path differs by platform:
    - Windows: %APPDATA%/Godot/app_userdata/YourGameName/
    - macOS: ~/Library/Application Support/Godot/app_userdata/YourGameName/
    - Linux: ~/.local/share/godot/app_userdata/YourGameName/
- On export, this is where you can **create, write, and update files at runtime**.
- Players always have write permission here.
- This is your proper place for:
    - Save games
    - Player profiles
    - Config settings (keybindings, graphics options)
    - Logs, caches, etc.

**Usage:** everything **mutable at runtime** that must persist between sessions.



TLDR:
- DeckConfig (**Resource**, in `res://`): editor-authored list of **CardConfig ids** (or weighted entries or `.tres` resources created from `.gd` resource class defenition) for starters/recipes.
- ProfileSave (**Resource**, written to `user://`) for persistent ownership/progress.
- for the local scoped like current runtime state of the board game states like health of the card just a regular script with fields in it