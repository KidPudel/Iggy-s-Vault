# Godot to Unity Translation

> Quick reference for translating Godot concepts to Unity equivalents.

---

## Core Concepts

| Godot                 | Unity                                             |
| --------------------- | ------------------------------------------------- |
| Node                  | GameObject                                        |
| Script (extends Node) | MonoBehaviour (component)                         |
| Scene (.tscn)         | Prefab (.prefab)                                  |
| Resource (.tres)      | ScriptableObject (.asset)                         |
| Autoload              | Singleton + DontDestroyOnLoad or Persistent Scene |
| Signal                | C# event / UnityEvent                             |
| Group                 | Tag / Layer                                       |
| PackedScene           | Prefab reference                                  |
| RefCounted            | Plain C# class                                    |

---

## Lifecycle Methods

| Godot                     | Unity                         |
| ------------------------- | ----------------------------- |
| `_init()`                 | `Awake()` (avoid constructor) |
| `_enter_tree()`           | `Awake()`                     |
| `_ready()`                | `Start()` (once ever)         |
| `_process(delta)`         | `Update()`                    |
| `_physics_process(delta)` | `FixedUpdate()`               |
| `_exit_tree()`            | `OnDestroy()`                 |
| —                         | `OnEnable()` / `OnDisable()`  |

---

## Scene Tree / Hierarchy

|Godot|Unity|
|---|---|
|`get_parent()`|`transform.parent`|
|`get_children()`|`foreach (Transform child in transform)`|
|`add_child(node)`|`obj.transform.SetParent(parent)`|
|`remove_child(node)`|`obj.transform.SetParent(null)`|
|`get_node("Path/To/Node")`|`transform.Find("Path/To/Node")`|
|`$ChildName`|`[SerializeField]` or `transform.Find()`|
|`%UniqueName`|`[SerializeField]` or FindWithTag|
|`get_tree().root`|`SceneManager.GetActiveScene()`|

---

## Transform / Position

|Godot|Unity|
|---|---|
|`position`|`transform.localPosition`|
|`global_position`|`transform.position`|
|`rotation`|`transform.localEulerAngles`|
|`global_rotation`|`transform.eulerAngles`|
|`rotation_degrees`|`transform.eulerAngles`|
|`scale`|`transform.localScale`|

---

## Instantiation / Destruction

|Godot|Unity|
|---|---|
|`scene.instantiate()`|`Instantiate(prefab)`|
|`queue_free()`|`Destroy(gameObject)`|
|`free()`|`DestroyImmediate()` (avoid)|
|`is_instance_valid(node)`|`if (obj)` (implicit bool)|

---

## Signals / Events

|Godot|Unity|
|---|---|
|`signal my_signal`|`public event Action MyEvent;`|
|`emit_signal("my_signal")`|`MyEvent?.Invoke();`|
|`connect("signal", callable)`|`obj.MyEvent += Handler;`|
|`disconnect("signal", callable)`|`obj.MyEvent -= Handler;`|

---

## Export / Serialization

|Godot|Unity|
|---|---|
|`@export var x: int`|`[SerializeField] private int _x;`|
|`@export_range(0, 100)`|`[Range(0, 100)]`|
|`@export_enum("A", "B")`|Use `enum` type|
|`@export_group("Name")`|`[Header("Name")]`|
|`@export_file("*.png")`|Use specific type (`Sprite`)|

---

## Scene Management

|Godot|Unity|
|---|---|
|`get_tree().change_scene_to_file(path)`|`SceneManager.LoadScene()`|
|`get_tree().reload_current_scene()`|`SceneManager.LoadScene(current)`|
|Adding child scene|`LoadScene(_, LoadSceneMode.Additive)`|
|Autoload|Persistent scene (never unloaded)|

---

## Input

| Godot                            | Unity (old)             | Unity (new Input System)       |
| -------------------------------- | ----------------------- | ------------------------------ |
| `Input.is_action_pressed()`      | `Input.GetButton()`     | `action.IsPressed()`           |
| `Input.is_action_just_pressed()` | `Input.GetButtonDown()` | `action.WasPressedThisFrame()` |
| `Input.get_axis()`               | `Input.GetAxis()`       | `action.ReadValue<float>()`    |

---

## Physics

|Godot|Unity|
|---|---|
|`CharacterBody3D`|`CharacterController` or Rigidbody|
|`RigidBody3D`|`Rigidbody`|
|`Area3D`|`Collider` (isTrigger = true)|
|`move_and_slide()`|`CharacterController.Move()`|
|`_physics_process(delta)`|`FixedUpdate()`|

---

## Collision Callbacks

|Godot|Unity|
|---|---|
|`body_entered` (Area)|`OnTriggerEnter()`|
|`body_exited` (Area)|`OnTriggerExit()`|
|`_on_body_entered` (physics)|`OnCollisionEnter()`|

---

## Common Operations

|Operation|Godot|Unity|
|---|---|---|
|Get component|Script on node|`GetComponent<T>()`|
|Add component|Add script to node|`AddComponent<T>()`|
|Find by type|`get_tree().get_nodes_in_group()`|`FindObjectOfType<T>()`|
|Find by name|`get_node()`|`GameObject.Find()`|
|Find by tag|Groups|`FindWithTag()`|
|Spawn|`instantiate()`|`Instantiate()`|
|Destroy|`queue_free()`|`Destroy()`|
|Delay|`await get_tree().create_timer()`|`yield return new WaitForSeconds()`|
|Print|`print()`|`Debug.Log()`|

---

## Key Behavioral Differences

|Aspect|Godot|Unity|
|---|---|---|
|`_ready()` timing|Each tree entry|Once ever (`Start`)|
|Node references|Path-based (`$`, `get_node`)|Serialized or GetComponent|
|Script per node|Typically one|Multiple components normal|
|Inheritance model|Node type + composition|Pure composition|
|Signals|Built-in to every Object|Must declare yourself|
|Execution order|Tree-based (mostly)|Undefined by default|
|Resource mutation|Common (with care)|Avoid (SO persists in Editor)|

---

See specific documents for details:

- [[MonoBehaviour Reference]]
- [[Communication Patterns in Unity]]
- [[Execution Order in Unity]]
- [[Unity Gotchas for Godot Developers]]