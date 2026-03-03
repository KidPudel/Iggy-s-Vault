# Unity Input System Implementation

Using Unity Input Actions in code via Generated C# Class and PlayerInput Component workflows.

## Sources
- [Unity Manual: Input System](https://docs.unity3d.com/Packages/com.unity.inputsystem@1.7/manual/index.html)
- [Unity API: InputAction.CallbackContext](https://docs.unity3d.com/Packages/com.unity.inputsystem@1.7/api/UnityEngine.InputSystem.InputAction.CallbackContext.html)
- [Unity API: PlayerInput](https://docs.unity3d.com/Packages/com.unity.inputsystem@1.7/api/UnityEngine.InputSystem.PlayerInput.html)
- [Unity API: InputActionRebindingExtensions](https://docs.unity3d.com/Packages/com.unity.inputsystem@1.7/api/UnityEngine.InputSystem.InputActionRebindingExtensions.html)

## Related topics
- [[Unity Input System Setup]]
- [[Input System in Unity]]
- [[Csharp Advanced Features Reference]]
- [[Communication Patterns in Unity]]
- [[MonoBehaviour Reference]]
- [[Unity Index Book of References]]

---

## Required Using Directives

```csharp
using UnityEngine.InputSystem;
```

## Generated C# Class Workflow

Given an Input Action Asset named `PlayerControls` with Action Map `Player` containing actions `Move` (Value, Vector2) and `Jump` (Button):

### Setup and Teardown

```csharp
public class PlayerController : MonoBehaviour
{
    private PlayerControls controls;

    void Awake()
    {
        controls = new PlayerControls();
    }

    void OnEnable()
    {
        controls.Player.Enable();
    }

    void OnDisable()
    {
        controls.Player.Disable();
    }
}
```

### Subscribing to Callbacks

```csharp
void Awake()
{
    controls = new PlayerControls();

    controls.Player.Jump.performed += OnJump;
    controls.Player.Jump.canceled += OnJumpCanceled;
    controls.Player.Jump.started += OnJumpStarted;
}

void OnDestroy()
{
    controls.Player.Jump.performed -= OnJump;
    controls.Player.Jump.canceled -= OnJumpCanceled;
    controls.Player.Jump.started -= OnJumpStarted;
}

private void OnJump(InputAction.CallbackContext context)
{
    // context.phase == InputActionPhase.Performed
}
```

### Polling in Update

```csharp
void Update()
{
    Vector2 moveInput = controls.Player.Move.ReadValue<Vector2>();
    bool jumpPressed = controls.Player.Jump.WasPressedThisFrame();
    bool jumpReleased = controls.Player.Jump.WasReleasedThisFrame();
    bool jumpHeld = controls.Player.Jump.IsPressed();
}
```

## PlayerInput Component Workflow

### Send Messages / Broadcast Messages

Method names are `On` + action name. Must be `public` or Unity cannot find them.

```csharp
public class PlayerController : MonoBehaviour
{
    // Called by PlayerInput via SendMessage
    public void OnMove(InputValue value)
    {
        Vector2 moveInput = value.Get<Vector2>();
    }

    public void OnJump(InputValue value)
    {
        // value.isPressed for button state
        bool pressed = value.isPressed;
    }

    public void OnLook(InputValue value)
    {
        Vector2 lookInput = value.Get<Vector2>();
    }
}
```

### Invoke Unity Events

Wire up in Inspector. Callback signature uses `InputAction.CallbackContext`:

```csharp
public void OnMove(InputAction.CallbackContext context)
{
    Vector2 moveInput = context.ReadValue<Vector2>();
}

public void OnJump(InputAction.CallbackContext context)
{
    if (context.performed)
    {
        // jump
    }
}
```

### Invoke C# Events

```csharp
private PlayerInput playerInput;

void Awake()
{
    playerInput = GetComponent<PlayerInput>();
    playerInput.onActionTriggered += OnActionTriggered;
}

void OnDestroy()
{
    playerInput.onActionTriggered -= OnActionTriggered;
}

private void OnActionTriggered(InputAction.CallbackContext context)
{
    if (context.action.name == "Move")
    {
        Vector2 moveInput = context.ReadValue<Vector2>();
    }
}
```

## InputAction.CallbackContext

| Member | Type | Description |
|---|---|---|
| `action` | `InputAction` | The action that triggered the callback |
| `control` | `InputControl` | The specific control that triggered it |
| `phase` | `InputActionPhase` | `Started`, `Performed`, or `Canceled` |
| `time` | `double` | Time the input occurred |
| `startTime` | `double` | Time the action entered `Started` |
| `duration` | `double` | `time - startTime` |
| `interaction` | `IInputInteraction` | The interaction driving the action (if any) |
| `performed` | `bool` | Shorthand for `phase == Performed` |
| `started` | `bool` | Shorthand for `phase == Started` |
| `canceled` | `bool` | Shorthand for `phase == Canceled` |
| `ReadValue<TValue>()` | `TValue` | Read the action's value as type `TValue` |
| `ReadValueAsButton()` | `bool` | Read value as bool (> default press threshold) |

## InputActionPhase

| Phase | Meaning |
|---|---|
| `Disabled` | Action is disabled |
| `Waiting` | Action is enabled, no input |
| `Started` | Input has begun (e.g., button pressed down) |
| `Performed` | Input completed (modified by Interactions) |
| `Canceled` | Input was interrupted or released |

## InputAction Properties and Methods

| Member | Signature | Description |
|---|---|---|
| `ReadValue<TValue>()` | `TValue ReadValue<TValue>()` | Current value |
| `IsPressed()` | `bool IsPressed()` | Currently actuated above press threshold |
| `WasPressedThisFrame()` | `bool WasPressedThisFrame()` | Pressed this frame |
| `WasReleasedThisFrame()` | `bool WasReleasedThisFrame()` | Released this frame |
| `WasPerformedThisFrame()` | `bool WasPerformedThisFrame()` | `performed` fired this frame |
| `triggered` | `bool` | True if `performed` this frame |
| `phase` | `InputActionPhase` | Current phase |
| `activeControl` | `InputControl` | Control currently driving value |
| `Enable()` | `void Enable()` | Enable the action |
| `Disable()` | `void Disable()` | Disable the action |
| `Reset()` | `void Reset()` | Reset action state |

## InputValue (Send/Broadcast Messages)

| Member | Signature | Description |
|---|---|---|
| `Get<TValue>()` | `TValue Get<TValue>()` | Read value |
| `Get()` | `object Get()` | Read value as object |
| `isPressed` | `bool` | Button state |

## Common Action Types and ReadValue Types

| Action Purpose | Action Type | ReadValue Type | Example |
|---|---|---|---|
| Movement | Value / Vector2 | `ReadValue<Vector2>()` | WASD, left stick |
| Look/Aim | Value / Vector2 | `ReadValue<Vector2>()` | Mouse delta, right stick |
| Jump/Fire | Button | `ReadValueAsButton()` or check `performed` | Spacebar, A button |
| Trigger | Value / Axis | `ReadValue<float>()` | Analog trigger |
| Scroll | Value / Vector2 | `ReadValue<Vector2>()` | Mouse scroll wheel |
| Selection | Value / Integer | `ReadValue<int>()` | Number keys |

## Action Map Switching

```csharp
// Via generated class
controls.Player.Disable();
controls.UI.Enable();

// Via PlayerInput component
playerInput.SwitchCurrentActionMap("UI");

// Current map name
string currentMap = playerInput.currentActionMap.name;
```

### PlayerInput Map Change Event

```csharp
playerInput.onControlsChanged += OnControlsChanged;

private void OnControlsChanged(PlayerInput input)
{
    // input.currentControlScheme -- "Keyboard&Mouse", "Gamepad", etc.
}
```

## Input Action Reference via Inspector

```csharp
[SerializeField] private InputActionReference moveActionRef;

void Update()
{
    Vector2 move = moveActionRef.action.ReadValue<Vector2>();
}
```

> [!warning]
> `InputActionReference` references actions in the asset. The action must be enabled (via PlayerInput or manually) to produce values.

## Direct InputAction in Script

```csharp
[SerializeField] private InputAction jumpAction;

void OnEnable()
{
    jumpAction.Enable();
}

void OnDisable()
{
    jumpAction.Disable();
}

void Update()
{
    if (jumpAction.WasPressedThisFrame()) { /* jump */ }
}
```

## Rebinding API

Namespace: `UnityEngine.InputSystem`

### Performing a Rebind

```csharp
InputActionRebindingExtensions.RebindingOperation rebindOp;

void StartRebind(InputAction action, int bindingIndex)
{
    action.Disable();

    rebindOp = action.PerformInteractiveRebinding(bindingIndex)
        .WithControlsExcluding("Mouse")          // exclude mouse
        .WithControlsExcluding("<Keyboard>/escape") // exclude specific
        .WithCancelingThrough("<Keyboard>/escape")  // cancel key
        .WithTargetBinding(bindingIndex)
        .OnMatchWaitForAnother(0.1f)              // de-bounce
        .OnComplete(operation => OnRebindComplete(operation, action))
        .OnCancel(operation => OnRebindCancel(operation, action))
        .Start();
}

void OnRebindComplete(InputActionRebindingExtensions.RebindingOperation op, InputAction action)
{
    op.Dispose();
    action.Enable();
}

void OnRebindCancel(InputActionRebindingExtensions.RebindingOperation op, InputAction action)
{
    op.Dispose();
    action.Enable();
}
```

### RebindingOperation Methods (Chaining)

| Method | Description |
|---|---|
| `.WithControlsExcluding(string path)` | Exclude controls matching path |
| `.WithControlsHavingToExclude(string path)` | Exclude controls (alternate) |
| `.WithCancelingThrough(string path)` | Binding path that cancels rebind |
| `.WithTargetBinding(int index)` | Which binding to rebind |
| `.WithExpectedControlType(string type)` | Expected control type ("Button", etc.) |
| `.OnMatchWaitForAnother(float seconds)` | Wait time after match before accepting |
| `.OnComplete(Action<RebindingOperation>)` | Callback on success |
| `.OnCancel(Action<RebindingOperation>)` | Callback on cancel |
| `.OnPotentialMatch(Action<RebindingOperation>)` | Callback on potential match |
| `.Start()` | Begin the rebind operation |
| `.Dispose()` | Dispose operation (required) |

### Saving / Loading Rebinds

```csharp
// Save all overrides as JSON
string rebinds = controls.asset.SaveBindingOverridesAsJson();
PlayerPrefs.SetString("rebinds", rebinds);

// Load overrides from JSON
string rebinds = PlayerPrefs.GetString("rebinds");
controls.asset.LoadBindingOverridesFromJson(rebinds);

// Remove all overrides
controls.asset.RemoveAllBindingOverrides();
```

### Display Binding Strings

```csharp
// Get human-readable binding name
string displayName = action.GetBindingDisplayString(bindingIndex);
// e.g., "Space", "A", "Left Stick"

// With specific display options
string displayName = action.GetBindingDisplayString(
    bindingIndex,
    InputBinding.DisplayStringOptions.DontUseShortDisplayNames
);
```
