# GDScript Type Checking Reference (Godot 4.x)

## Overview

GDScript is **progressively typed**: you can write fully dynamic code, add static types incrementally, or use both styles. Type hints are not just documentation—they're enforced at runtime and enable compile-time error detection.

---

## Core Type Checking Methods

### 1. `is` Keyword — Inheritance-Aware Check

Checks if an object is an instance of a class **or any of its subclasses**.

```gdscript
if item is PlantInstance:
    # True for PlantInstance and any class extending it
    pass
```

**Key behaviors:**

- Works with `class_name` registered classes
- Works with preloaded scripts (must be `const`)
- Checks the full inheritance chain
- Returns `true` if the object is the type OR a descendant

```gdscript
# With class_name
if obj is MyCustomClass:
    pass

# With preloaded script (MUST be const in Godot 4)
const DoorScript = preload("res://door.gd")
if area is DoorScript:
    pass
```

**Gotcha:** `is` doesn't work for primitive types like `String`, `int`, `float`. Use `typeof()` instead.

---

### 2. `typeof()` — Variant Type Check

Returns an integer constant from `Variant.Type` enum. Essential for primitives and unknown types.

```gdscript
if typeof(value) == TYPE_STRING:
    pass

if typeof(value) == TYPE_INT:
    pass

if typeof(value) == TYPE_OBJECT:
    # It's an object, use `is` or get_script() for specifics
    pass
```

__Common TYPE__ constants:_*

| Constant          | Value | Type       |
| ----------------- | ----- | ---------- |
| `TYPE_NIL`        | 0     | null       |
| `TYPE_BOOL`       | 1     | bool       |
| `TYPE_INT`        | 2     | int        |
| `TYPE_FLOAT`      | 3     | float      |
| `TYPE_STRING`     | 4     | String     |
| `TYPE_VECTOR2`    | 5     | Vector2    |
| `TYPE_VECTOR3`    | 9     | Vector3    |
| `TYPE_DICTIONARY` | 27    | Dictionary |
| `TYPE_ARRAY`      | 28    | Array      |
| `TYPE_OBJECT`     | 24    | Object     |

---

### 3. `get_script()` — Exact Type Comparison

Compare script references for **exact type matching** (excludes subclasses).

```gdscript
# Check if two objects are the EXACT same type
if item_a.get_script() == item_b.get_script():
    # Same exact class (not just related via inheritance)
    pass

# Compare against a known script
const PlantScript = preload("res://plant_instance.gd")
if item.get_script() == PlantScript:
    pass
```

**When to use:**

- Need exact type, not "is-a" relationship
- Comparing two objects' types at runtime
- Custom classes without `class_name`

---

### 4. `get_class()` / `is_class()` — Engine Class Check

Works for **built-in engine classes only** (Node, Resource, etc.). Custom scripts return the base engine class.

```gdscript
print(node.get_class())  # "Node2D", "CharacterBody2D", etc.

if node.is_class("CharacterBody2D"):
    pass

# Check inheritance chain for built-in classes
if ClassDB.is_parent_class("CharacterBody2D", node.get_class()):
    pass
```

**Limitation:** For a Node2D with a custom script, `get_class()` returns `"Node2D"`, not your class name.

---

### 5. `get_global_name()` — Script Class Name (Godot 4.3+)

Get the `class_name` string of a script at runtime.

```gdscript
var script = obj.get_script()
if script:
    var class_name_str = script.get_global_name()  # Returns "PlantInstance" etc.
```

---

## Exact vs Inheritance-Based Comparison

|Method|Checks Inheritance?|Use Case|
|---|---|---|
|`is`|✅ Yes|"Is this a Plant or subclass of Plant?"|
|`get_script() ==`|❌ No|"Is this exactly a PlantInstance?"|
|`typeof()`|N/A|"Is this a String/int/Array?"|
|`get_class()`|❌ No|"What engine class is this?"|

---

## Your Specific Problem Solved

```gdscript
# item_instance.gd
class_name ItemInstance
extends Resource

# plant_instance.gd  
class_name PlantInstance
extends ItemInstance

# seed_instance.gd
class_name SeedInstance
extends ItemInstance
```

### Check if Two Items Are Same Type

```gdscript
# EXACT type match (PlantInstance != SeedInstance, PlantInstance != subclass)
func are_same_exact_type(a: ItemInstance, b: ItemInstance) -> bool:
    return a.get_script() == b.get_script()

# INHERITANCE-AWARE (true if both PlantInstance or subclass of PlantInstance)
func are_both_plants(a: ItemInstance, b: ItemInstance) -> bool:
    return a is PlantInstance and b is PlantInstance
```

### In Your HotBarSystem

```gdscript
class_name HotBarSystem
extends Node

var current_item: ItemInstance

func _set_item(item: ItemInstance) -> void:
    # Check against current item's exact type
    if current_item and item.get_script() == current_item.get_script():
        # Same exact type - maybe stack them
        pass
    
    # Or check for specific type
    if item is PlantInstance:
        var plant := item as PlantInstance  # Safe cast with autocomplete
        # plant-specific logic
    elif item is SeedInstance:
        var seed := item as SeedInstance
        # seed-specific logic
```

---

## Static Type Hints Syntax

### Variable Declaration

```gdscript
var health: int = 100           # Explicit type
var speed := 5.0                # Inferred type (float)
var items: Array[ItemInstance]  # Typed array (Godot 4+)
```

### Function Signatures

```gdscript
func take_damage(amount: int) -> void:
    health -= amount

func get_inventory() -> Array[ItemInstance]:
    return items
```

### Type Casting with `as`

```gdscript
func _on_body_entered(body: Node2D) -> void:
    var player := body as Player  # Returns null if cast fails
    if player:
        player.take_damage(10)
```

**Cast behavior:**

- Custom class cast fails → returns `null`
- Built-in type cast fails → throws error

---

## Common Patterns

### Safe Type Narrowing

```gdscript
func process_item(item: ItemInstance) -> void:
    if item is PlantInstance:
        var plant := item as PlantInstance
        plant.grow()  # Full autocomplete available
```

### Type-Based Dispatch

```gdscript
func handle_item(item: ItemInstance) -> void:
    match item.get_script():
        PlantInstance:
            _handle_plant(item as PlantInstance)
        SeedInstance:
            _handle_seed(item as SeedInstance)
        _:
            _handle_generic(item)
```

### Runtime Type Name

```gdscript
func get_type_name(obj: Object) -> String:
    var script = obj.get_script()
    if script:
        var global_name = script.get_global_name()  # Godot 4.3+
        if global_name:
            return global_name
        return script.resource_path.get_file().get_basename()
    return obj.get_class()
```

---

## Quick Reference Table

|Goal|Method|
|---|---|
|Is X a Y or subclass?|`x is Y`|
|Is X exactly type Y?|`x.get_script() == Y` or `x.get_script() == preload("y.gd")`|
|Is X a primitive type?|`typeof(x) == TYPE_*`|
|Get engine class name|`x.get_class()`|
|Get script class name|`x.get_script().get_global_name()` (4.3+)|
|Safe downcast|`var y := x as Y` (null if fails)|
|Compare two objects' types|`a.get_script() == b.get_script()`|

---

## Warnings & Editor Settings

Enable these in Project Settings → Debug → GDScript:

- `UNTYPED_DECLARATION` — Warns on variables without type hints
- `INFERRED_DECLARATION` — Warns when using `:=` instead of explicit types
- `UNSAFE_*` — Warns on potentially unsafe operations

The editor shows **green line numbers** for type-safe lines (can be configured in Editor Settings).