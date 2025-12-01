`@export` tells the godot **to save** their value (any type) along with the [[Godot resource]] they are attached to, by adding metadata to a script. As well as exposed in the editor to set at the design time.
On scene load the engine restores the value before `_ready()`

- allows to change in inspector
- allows for serialization when saving and loading



# Example

```gdscript
@export var enemy_scene: PackedScene @export var hp: int = 100
@export var camera_target: Node
```
### **In Editor:**
- enemy_scene → lets you pick a .tscn file
- hp → shows an integer field
- camera_target → lets you drag any existing Node from the scene tree into this slot


# @export Decoration Variations

## Basic Types

```gdscript
@export var my_int: int
@export var my_float: float
@export var my_string: String
@export var my_bool: bool
```

## Ranges and Hints

```gdscript
@export_range(0, 100) var percentage: int
@export_range(0.0, 1.0, 0.01) var alpha: float  # With step
@export_range(0, 100, 5, "or_greater") var score: int  # Allows values > 100
@export_range(0, 100, 1, "or_less") var health: int  # Allows values < 0
```

## Enums

```gdscript
@export_enum("Warrior", "Mage", "Rogue") var character_class: String
@export_enum("Easy:0", "Normal:1", "Hard:2") var difficulty: int
```

## Flags (Bitmask)

```gdscript
@export_flags("Fire", "Water", "Earth", "Air") var elements: int
@export_flags("Run:1", "Jump:2", "Dash:4") var abilities: int
```

## Files and Directories

```gdscript
@export_file var any_file: String
@export_file("*.png", "*.jpg") var image_file: String
@export_file("*.tscn") var scene_file: String
@export_dir var folder_path: String
@export_global_file var global_file: String  # Not restricted to project
@export_global_dir var global_folder: String
```

## Resources

```gdscript
@export var texture: Texture2D
@export var scene: PackedScene
@export var material: Material
@export var custom_resource: MyCustomResource
```

## Colors

```gdscript
@export_color_no_alpha var color: Color
@export var color_with_alpha: Color  # Default includes alpha
```

## Multiline Text

```gdscript
@export_multiline var description: String
@export_multiline var dialogue: String
```

## Node Paths

```gdscript
@export var target_node: NodePath
@export var player_path: NodePath
```

## Arrays

```gdscript
@export var items: Array
@export var numbers: Array[int]
@export var scenes: Array[PackedScene]
@export var strings: Array[String]
```

## Grouping

```gdscript
@export_group("Movement")
@export var speed: float
@export var jump_height: float

@export_group("Combat")
@export var damage: int
@export var attack_speed: float

@export_subgroup("Defense")  # Nested under current group
@export var armor: int
@export var dodge_chance: float
```

## Categories

```gdscript
@export_category("Player Stats")  # Creates a new category
@export var health: int
@export var mana: int
```

## Placeholders

```gdscript
@export_placeholder("Enter player name...") var player_name: String
```

## Custom Property Hints (Advanced)

```gdscript
@export_custom(PROPERTY_HINT_NONE, "") var custom_var: int
```

## Common Combinations

```gdscript
@export_group("Spawn Settings")
@export var enemy_scene: PackedScene
@export_range(1, 10) var spawn_count: int
@export_range(0.5, 5.0, 0.1) var spawn_interval: float
@export_flags("Ground", "Air", "Water") var spawn_locations: int

@export_category("Audio")
@export var sound_effect: AudioStream
@export_range(-80, 24) var volume_db: float
```
