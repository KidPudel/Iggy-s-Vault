# Godot Reflection Reference

## What is Reflection?

Reflection is the ability to inspect and manipulate objects, methods, and properties at runtime using strings rather than direct references.

## Core Methods

### Method Reflection

```gdscript
# Check if a method exists
if object.has_method("method_name"):
    # Call it dynamically
    object.call("method_name", arg1, arg2)
    
# Call with array of arguments
object.callv("method_name", [arg1, arg2])

# Get list of all methods
var methods = object.get_method_list()
for method in methods:
    print(method.name)  # Method name
    print(method.args)  # Argument info
```

### Property Reflection

```gdscript
# Check if property exists
if "health" in object:
    # Get property value
    var value = object.get("health")
    
# Set property value
object.set("health", 100)

# Get list of all properties
var properties = object.get_property_list()
for prop in properties:
    print(prop.name)
    print(prop.type)
```

### Signal Reflection

```gdscript
# Check if signal exists
if object.has_signal("damaged"):
    # Connect dynamically
    object.connect("damaged", callable)
    
# Get list of signals
var signals = object.get_signal_list()
```

## Common Use Cases

### 1. Command Pattern / Action System

```gdscript
func execute_action(obj: Node, action: String):
    if obj.has_method(action):
        obj.call(action)
    else:
        push_error("Action not found: " + action)
```

### 2. Save/Load System

```gdscript
func save_object(obj: Object) -> Dictionary:
    var data = {}
    for prop in obj.get_property_list():
        if prop.usage & PROPERTY_USAGE_SCRIPT_VARIABLE:
            data[prop.name] = obj.get(prop.name)
    return data

func load_object(obj: Object, data: Dictionary):
    for key in data:
        if key in obj:
            obj.set(key, data[key])
```

### 3. Generic Event Handler

```gdscript
func trigger_event(target: Node, event_name: String, params: Array = []):
    var method = "on_" + event_name
    if target.has_method(method):
        target.callv(method, params)
```

## Performance Considerations

**Reflection is slower than direct calls:**

- `call()` is ~3-10x slower than direct method calls
- `get()`/`set()` is ~2-5x slower than direct property access
- Use for initialization, serialization, or infrequent actions
- Avoid in per-frame logic (`_process`, `_physics_process`)

## When to Use Reflection

✅ **Good for:**

- Save/load systems
- Plugin architectures
- Data-driven systems
- Tool scripts and editor utilities
- Infrequent runtime decisions

❌ **Avoid for:**

- Per-frame updates
- Performance-critical code
- When you know the types at compile time

## When NOT to Use Reflection

Use direct calls or interfaces instead:

```gdscript
# Instead of this:
if enemy.has_method("take_damage"):
    enemy.call("take_damage", 10)

# Do this:
if enemy is Damageable:
    enemy.take_damage(10)
```

## Type Safety Alternative

Create interfaces to get autocomplete and error checking:

```gdscript
class_name Interactable extends RefCounted

func interact() -> void:
    pass

# Then use:
func use_object(obj: Interactable):
    obj.interact()  # Autocomplete works, errors if method missing
```

## Debug Tips

```gdscript
# Print all methods for debugging
func debug_object(obj: Object):
    print("=== Methods ===")
    for method in obj.get_method_list():
        print(method.name)
    
    print("=== Properties ===")
    for prop in obj.get_property_list():
        if prop.usage & PROPERTY_USAGE_SCRIPT_VARIABLE:
            print("%s = %s" % [prop.name, obj.get(prop.name)])
```

## Key Takeaway

Reflection trades compile-time safety and performance for runtime flexibility. Use it when you need dynamic behavior, but prefer typed approaches for most gameplay code.