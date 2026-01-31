# Input System

> Unity's new input system. Action-based, supports rebinding, multiple devices.

**Package:** `com.unity.inputsystem`

---

## Setup

1. Package Manager → Install "Input System"
2. Edit → Project Settings → Player → Active Input Handling → **Both** (or Input System Package)
3. Create Input Actions asset: Right-click → Create → Input Actions

---

## Core Concepts

|Concept|What It Is|
|---|---|
|**Input Actions Asset**|Container for all your input mappings (.inputactions file)|
|**Action Map**|Group of actions (e.g., "Player", "UI", "Vehicle")|
|**Action**|Single input (e.g., "Jump", "Move", "Fire")|
|**Binding**|What triggers the action (Space key, A button, etc.)|
|**Control Scheme**|Device grouping (Keyboard+Mouse, Gamepad)|

```
InputActions.inputactions
├── Player (Action Map)
│   ├── Move (Action) → WASD, Left Stick
│   ├── Jump (Action) → Space, South Button
│   └── Fire (Action) → Left Click, Right Trigger
└── UI (Action Map)
    ├── Navigate → Arrow Keys, D-Pad
    └── Submit → Enter, South Button
```

---

## Action Types

|Type|Returns|Use For|
|---|---|---|
|**Value**|Continuous value|Movement, look, triggers|
|**Button**|Press state|Jump, fire, interact|
|**Pass Through**|All inputs (no conflict resolution)|Multiple players, debug|

---

## Reading Input

### Option 1: PlayerInput Component (Easiest)

Add `PlayerInput` component to GameObject, assign Input Actions asset.

**Send Messages** (default):

```csharp
public class Player : MonoBehaviour
{
    // Method names must match action names: "On" + ActionName
    public void OnMove(InputValue value)
    {
        Vector2 input = value.Get<Vector2>();
    }
    
    public void OnJump(InputValue value)
    {
        if (value.isPressed)
            Jump();
    }
    
    public void OnFire(InputValue value)
    {
        // value.isPressed for button down
    }
}
```

**Invoke Unity Events:**

- Set Behavior to "Invoke Unity Events"
- Wire up in Inspector like UI buttons

**Invoke C# Events:**

```csharp
PlayerInput playerInput;

void Awake()
{
    playerInput = GetComponent<PlayerInput>();
    playerInput.onActionTriggered += OnAction;
}

void OnAction(InputAction.CallbackContext context)
{
    if (context.action.name == "Jump" && context.performed)
        Jump();
}
```

---

### Option 2: Generated C# Class (Recommended for Code)

1. Select Input Actions asset
2. Inspector → Generate C# Class → Apply

```csharp
public class Player : MonoBehaviour
{
    private PlayerInputActions _input; // Generated class name
    
    void Awake()
    {
        _input = new PlayerInputActions();
    }
    
    void OnEnable()
    {
        _input.Player.Enable(); // Enable the "Player" action map
        
        _input.Player.Jump.performed += OnJump;
        _input.Player.Jump.canceled += OnJumpReleased;
    }
    
    void OnDisable()
    {
        _input.Player.Jump.performed -= OnJump;
        _input.Player.Jump.canceled -= OnJumpReleased;
        
        _input.Player.Disable();
    }
    
    void Update()
    {
        // Continuous reading
        Vector2 move = _input.Player.Move.ReadValue<Vector2>();
        Move(move);
    }
    
    void OnJump(InputAction.CallbackContext context)
    {
        Jump();
    }
    
    void OnJumpReleased(InputAction.CallbackContext context)
    {
        // Released
    }
}
```

---

### Option 3: Direct Device Access

No Input Actions asset needed. Quick and dirty.

```csharp
using UnityEngine.InputSystem;

void Update()
{
    var keyboard = Keyboard.current;
    if (keyboard == null) return;
    
    if (keyboard.spaceKey.wasPressedThisFrame)
        Jump();
    
    if (keyboard.wKey.isPressed)
        MoveForward();
    
    var mouse = Mouse.current;
    Vector2 mouseDelta = mouse.delta.ReadValue();
    Vector2 mousePos = mouse.position.ReadValue();
    
    if (mouse.leftButton.wasPressedThisFrame)
        Fire();
    
    var gamepad = Gamepad.current;
    if (gamepad != null)
    {
        Vector2 stick = gamepad.leftStick.ReadValue();
        float trigger = gamepad.rightTrigger.ReadValue();
        
        if (gamepad.buttonSouth.wasPressedThisFrame) // A on Xbox
            Jump();
    }
}
```

---

## Callback Context

```csharp
void OnJump(InputAction.CallbackContext context)
{
    // Phase
    if (context.started)  { }   // Just pressed
    if (context.performed) { }  // Fully pressed (after hold time if set)
    if (context.canceled) { }   // Released
    
    // Read value
    float value = context.ReadValue<float>();
    Vector2 vec = context.ReadValue<Vector2>();
    
    // Timing
    double time = context.time;
    double duration = context.duration;
    
    // What triggered it
    InputControl control = context.control;
    InputDevice device = context.control.device;
}
```

---

## Common Bindings

### Composite Bindings (WASD → Vector2)

In Input Actions editor:

1. Add Action, set type to **Value**, control type to **Vector2**
2. Add Binding → Add 2D Vector Composite
3. Assign Up/Down/Left/Right

```
Move (Value, Vector2)
├── WASD [2D Vector]
│   ├── Up: W
│   ├── Down: S
│   ├── Left: A
│   └── Right: D
└── Left Stick [Binding]
```

### Modifiers

```
Sprint (Button)
└── Left Shift [Binding]
    └── Hold (Interaction) - time: 0.2
    
Fire (Button)  
└── Left Click [Binding]
    └── Tap (Interaction) - for quick press only
```

---

## Interactions

|Interaction|Triggers When|
|---|---|
|**Press**|On press (default)|
|**Hold**|Held for duration|
|**Tap**|Pressed and released quickly|
|**SlowTap**|Held briefly then released|
|**MultiTap**|Double/triple tap|

```csharp
// Hold example: performed fires after hold time
_input.Player.Interact.performed += ctx => 
{
    // Only fires after holding for set duration
    InteractLong();
};

_input.Player.Interact.canceled += ctx =>
{
    // Released before hold completed
};
```

---

## Switching Action Maps

```csharp
// Disable one, enable another
_input.Player.Disable();
_input.UI.Enable();

// Or with PlayerInput component
playerInput.SwitchCurrentActionMap("UI");
```

Common pattern:

```csharp
void OpenMenu()
{
    _input.Player.Disable();
    _input.UI.Enable();
    Time.timeScale = 0;
}

void CloseMenu()
{
    _input.UI.Disable();
    _input.Player.Enable();
    Time.timeScale = 1;
}
```

---

## Rebinding

```csharp
private InputActionRebindingExtensions.RebindingOperation _rebindOp;

public void StartRebind(InputAction action)
{
    action.Disable();
    
    _rebindOp = action.PerformInteractiveRebinding()
        .WithControlsExcluding("Mouse")  // Optional filters
        .OnMatchWaitForAnother(0.1f)     // Wait for modifier release
        .OnComplete(operation => 
        {
            operation.Dispose();
            action.Enable();
            SaveBindings();
        })
        .OnCancel(operation =>
        {
            operation.Dispose();
            action.Enable();
        })
        .Start();
}

public void CancelRebind()
{
    _rebindOp?.Cancel();
}
```

### Save/Load Bindings

```csharp
// Save
string bindings = _input.asset.SaveBindingOverridesAsJson();
PlayerPrefs.SetString("InputBindings", bindings);

// Load
string bindings = PlayerPrefs.GetString("InputBindings", string.Empty);
if (!string.IsNullOrEmpty(bindings))
    _input.asset.LoadBindingOverridesFromJson(bindings);
```

---

## Multiple Players (Local)

```csharp
// PlayerInputManager handles player joining
// Add PlayerInputManager component to scene

PlayerInputManager.instance.onPlayerJoined += OnPlayerJoined;

void OnPlayerJoined(PlayerInput player)
{
    int index = player.playerIndex; // 0, 1, 2...
    InputDevice device = player.devices[0];
}
```

Or manual:

```csharp
// Assign specific device to player
PlayerInput.Instantiate(playerPrefab, 
    controlScheme: "Gamepad", 
    pairWithDevice: Gamepad.all[1]);
```

---

## Device Detection

```csharp
// Current device changed
InputSystem.onActionChange += (obj, change) =>
{
    if (change == InputActionChange.ActionPerformed)
    {
        var action = (InputAction)obj;
        var device = action.activeControl.device;
        
        if (device is Gamepad)
            ShowGamepadIcons();
        else if (device is Keyboard || device is Mouse)
            ShowKeyboardIcons();
    }
};

// Device connected/disconnected
InputSystem.onDeviceChange += (device, change) =>
{
    if (change == InputDeviceChange.Added)
        Debug.Log($"Device added: {device.displayName}");
    if (change == InputDeviceChange.Removed)
        Debug.Log($"Device removed: {device.displayName}");
};
```

---

## UI Integration

For UI navigation with new Input System:

1. Replace `StandaloneInputModule` with `InputSystemUIInputModule`
2. Assign Input Actions asset to it
3. UI actions map automatically to Navigate, Submit, Cancel, etc.

```csharp
// Cursor lock with UI
void Update()
{
    if (_input.UI.Cancel.WasPressedThisFrame())
    {
        if (Cursor.lockState == CursorLockMode.Locked)
        {
            Cursor.lockState = CursorLockMode.None;
            Cursor.visible = true;
        }
    }
}
```

---

## Common Gotchas

|Issue|Cause|Fix|
|---|---|---|
|Input not working|Action map not enabled|Call `.Enable()` on action map|
|Input not working in build|Wrong Active Input Handling|Project Settings → Both or Input System|
|`Keyboard.current` is null|No keyboard connected / wrong backend|Check for null, ensure Input System active|
|Events fire multiple times|Subscribed multiple times|Unsubscribe in `OnDisable`|
|UI not responding|Wrong input module|Use `InputSystemUIInputModule`|
|Rebind shows "Button South"|Raw binding path displayed|Use `GetBindingDisplayString()`|

---

## Quick Reference

```csharp
// Button states (direct device)
keyboard.spaceKey.wasPressedThisFrame  // Just pressed
keyboard.spaceKey.wasReleasedThisFrame // Just released  
keyboard.spaceKey.isPressed            // Currently held

// Reading values
action.ReadValue<float>()
action.ReadValue<Vector2>()
action.IsPressed()
action.WasPressedThisFrame()
action.WasReleasedThisFrame()

// Action phases
context.started    // Input started
context.performed  // Input performed (main callback)
context.canceled   // Input stopped

// Enable/disable
actionMap.Enable()
actionMap.Disable()
action.Enable()
action.Disable()

// Display names for UI
action.GetBindingDisplayString() // "Space" or "A"
```

---

## Godot Comparison

| Godot                            | Unity Input System                  |
| -------------------------------- | ----------------------------------- |
| Input Map (Project Settings)     | Input Actions Asset                 |
| `Input.is_action_pressed()`      | `action.IsPressed()`                |
| `Input.is_action_just_pressed()` | `action.WasPressedThisFrame()`      |
| `Input.get_vector()`             | `action.ReadValue<Vector2>()`       |
| Action name strings              | Generated C# class with type safety |
| Built-in                         | Requires package install            |