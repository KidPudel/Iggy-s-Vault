# Unity Input System

> **Action-based input** that supports rebinding, multiple devices, and control scheme switching. Replaces the old `Input.GetKey()` API.

**Package:** `com.unity.inputsystem`

---

## Why Use This

| Old Input System                     | New Input System                          |
| ------------------------------------ | ----------------------------------------- |
| Hard-coded key names                 | Action names (rebindable)                 |
| `if (Input.GetKey(KeyCode.Space))`   | `if (_input.Player.Jump.triggered)`       |
| Device-specific code                 | Automatic device switching                |
| Manual rebinding logic               | Built-in rebinding                        |
| No control schemes                   | Switch between KB+M, Gamepad, Touch       |

Use the new system when you need rebinding, gamepad support, or multi-device games. Stick with old system for prototypes or single-platform projects.

---

## Initial Setup

### 1. Install Package

Package Manager → Input System

### 2. Enable Input System

Edit → Project Settings → Player → Active Input Handling → **Input System Package** (or Both during migration)

Unity will restart.

### 3. Create Input Actions Asset

Right-click in Project → Create → Input Actions

Name it (e.g., `PlayerControls`). This creates a `.inputactions` file.

---

## Define Your Actions

Double-click the `.inputactions` asset to open the editor. You'll see three columns:

**Action Maps** (left column) → Groups of related actions  
**Actions** (middle column) → What the player does  
**Bindings** (right column) → Physical inputs that trigger it

### Example Setup

~~~
PlayerControls.inputactions
├── Player (Action Map)
│   ├── Move (Action)
│   │   └── Type: Value, Control Type: Vector2
│   ├── Jump (Action)
│   │   └── Type: Button
│   └── Fire (Action)
│       └── Type: Button
└── UI (Action Map)
    ├── Navigate (Action)
    └── Submit (Action)
~~~

### Add Bindings

1. Select the **Move** action
2. Click **+** → Add 2D Vector Composite → WASD
3. Expand the composite, assign:
   - Up: W
   - Down: S
   - Left: A
   - Right: D
4. Click **+** again → Add Binding → Left Stick (for gamepad)

5. Select the **Jump** action
6. Click **+** → Add Binding → Space
7. Click **+** → Add Binding → Button South (gamepad A button)

### Action Types

- **Button:** On/off input (Jump, Fire, Interact)
- **Value:** Continuous input (Move, Look, Trigger pressure)
- **Pass Through:** All inputs fire without conflict resolution (rare)

**Save** the asset (Ctrl+S).

---

## Choose Your Workflow

There are **two completely different ways** to use Input Actions. Pick one:

---

# Workflow A: Generated C# Class (Recommended)

**Best for:** Programmers who want full control and type safety.

## Step 1: Generate the Class

1. Select your `.inputactions` asset
2. In Inspector: check **Generate C# Class**
3. Click **Apply**

Unity creates a `.cs` file with the same name as your asset.

## Step 2: Use in Code

~~~csharp
using UnityEngine;
using UnityEngine.InputSystem;

public class PlayerController : MonoBehaviour
{
    private PlayerControls _input; // Name matches your .inputactions file
    private Vector2 _moveInput;
    
    private void Awake()
    {
        _input = new PlayerControls(); // Instantiate the generated class
    }
    
    private void OnEnable()
    {
        _input.Player.Enable(); // Enable the "Player" action map
        
        // Subscribe to button events
        _input.Player.Jump.performed += OnJump;
        _input.Player.Fire.performed += OnFire;
    }
    
    private void OnDisable()
    {
        // Always unsubscribe
        _input.Player.Jump.performed -= OnJump;
        _input.Player.Fire.performed -= OnFire;
        
        _input.Player.Disable();
    }
    
    private void Update()
    {
        // Read continuous input every frame
        _moveInput = _input.Player.Move.ReadValue<Vector2>();
        
        if (_moveInput != Vector2.zero)
            Move(_moveInput);
    }
    
    private void OnJump(InputAction.CallbackContext context)
    {
        Jump();
    }
    
    private void OnFire(InputAction.CallbackContext context)
    {
        Fire();
    }
    
    private void Move(Vector2 direction)
    {
        // Your movement code
    }
    
    private void Jump()
    {
        // Your jump code
    }
    
    private void Fire()
    {
        // Your fire code
    }
}
~~~

## Key Points

**No component in Inspector.** The generated class is just C# code.

**Two input patterns:**
- **Continuous** (move, look): Read in `Update()` with `ReadValue<T>()`
- **Event-driven** (jump, fire): Subscribe to `.performed` callback

**Enable/Disable:**
- Enable in `OnEnable()`, disable in `OnDisable()`
- This ensures proper cleanup when object is destroyed

**Action map structure:**
~~~csharp
_input.Player.Move    // "Player" is the action map name
_input.Player.Jump
_input.UI.Navigate    // "UI" is another action map
~~~

---

# Workflow B: PlayerInput Component (Alternative)

**Best for:** Quick setup, designers, or when you want Inspector configuration.

## Step 1: Add Component

1. Select your player GameObject
2. Add Component → Player Input
3. Assign your `.inputactions` asset to **Actions** field
4. Set **Behavior** to **Send Messages**

## Step 2: Write Handler Methods

~~~csharp
using UnityEngine;
using UnityEngine.InputSystem;

public class Player : MonoBehaviour
{
    // Method names MUST match action names with "On" prefix
    
    public void OnMove(InputValue value)
    {
        Vector2 input = value.Get<Vector2>();
        Move(input);
    }
    
    public void OnJump(InputValue value)
    {
        if (value.isPressed)
            Jump();
    }
    
    public void OnFire(InputValue value)
    {
        if (value.isPressed)
            Fire();
    }
    
    private void Move(Vector2 direction)
    {
        // Your movement code
    }
    
    private void Jump()
    {
        // Your jump code
    }
    
    private void Fire()
    {
        // Your fire code
    }
}
~~~

## Key Points

**Component required.** The `PlayerInput` component must be on the same GameObject as your script.

**Method naming convention:**
- Action named "Move" → Method named `OnMove`
- Action named "Jump" → Method named `OnJump`
- Must be public

**Unity calls these automatically** when input happens.

**Downsides:**
- String-based method names (typos won't show errors)
- Harder to refactor
- Less control over when actions are enabled

---

# Common Patterns (Works with Both Workflows)

## Switching Action Maps

When opening a menu, disable Player controls and enable UI controls:

**Generated Class Approach:**
~~~csharp
private void OpenMenu()
{
    _input.Player.Disable();
    _input.UI.Enable();
    _pauseMenu.SetActive(true);
}

private void CloseMenu()
{
    _input.UI.Disable();
    _input.Player.Enable();
    _pauseMenu.SetActive(false);
}
~~~

**PlayerInput Component Approach:**
~~~csharp
private PlayerInput _playerInput;

private void Awake()
{
    _playerInput = GetComponent<PlayerInput>();
}

private void OpenMenu()
{
    _playerInput.SwitchCurrentActionMap("UI");
    _pauseMenu.SetActive(true);
}

private void CloseMenu()
{
    _playerInput.SwitchCurrentActionMap("Player");
    _pauseMenu.SetActive(false);
}
~~~

---

## Action Callbacks (Generated Class Only)

Actions have three callback phases:

~~~csharp
_input.Player.Jump.started += OnJumpStarted;      // Input began
_input.Player.Jump.performed += OnJumpPerformed;  // Input completed (use this most)
_input.Player.Jump.canceled += OnJumpCanceled;    // Input released
~~~

**Most of the time, use `.performed`.**

**Use `.canceled` for:**
- Variable jump height (hold longer = jump higher)
- Charge attacks (release to fire)

~~~csharp
private bool _isJumping;

private void OnEnable()
{
    _input.Player.Jump.performed += ctx => _isJumping = true;
    _input.Player.Jump.canceled += ctx => _isJumping = false;
}
~~~

### Callback Context

~~~csharp
private void OnJump(InputAction.CallbackContext context)
{
    // Read value
    float value = context.ReadValue<float>();
    Vector2 vec = context.ReadValue<Vector2>();
    bool pressed = context.ReadValueAsButton();
    
    // Check phase
    if (context.started) { /* Just started */ }
    if (context.performed) { /* Main callback */ }
    if (context.canceled) { /* Released */ }
    
    // Timing
    double time = context.time;           // When input occurred
    double duration = context.duration;   // How long held
    
    // Device info
    InputDevice device = context.control.device;
}
~~~

---

## Interactions

Add interactions to change when `.performed` fires:

| Interaction | Performed Fires                          | Use For              |
| ----------- | ---------------------------------------- | -------------------- |
| Press       | On press (default)                       | Standard button      |
| **Hold**    | After held for duration                  | Charge attacks       |
| **Tap**     | Pressed and released quickly             | Double-tap dash      |
| **Multi Tap**| Multiple quick presses                  | Combo inputs         |

**How to add:**
1. Open `.inputactions` editor
2. Select a binding (e.g., Space for Jump)
3. Click **+** → Add Interaction → Hold
4. Set duration (e.g., 0.5 seconds)

**Code:**
~~~csharp
// performed now only fires after holding for 0.5s
_input.Player.Interact.performed += ctx => StartInteraction();

// canceled fires if released early
_input.Player.Interact.canceled += ctx => CancelInteraction();
~~~

---

## Rebinding

~~~csharp
using UnityEngine.InputSystem;

public class RebindManager : MonoBehaviour
{
    private PlayerControls _input;
    private InputActionRebindingExtensions.RebindingOperation _rebindOp;
    
    public void StartRebind(InputAction action, int bindingIndex)
    {
        action.Disable();
        
        _rebindOp = action.PerformInteractiveRebinding(bindingIndex)
            .WithControlsExcluding("Mouse")  // Ignore mouse movement
            .OnMatchWaitForAnother(0.1f)     // Wait for modifiers
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
    
    // Display current binding for UI
    public string GetBindingText(InputAction action, int bindingIndex)
    {
        return action.GetBindingDisplayString(bindingIndex);
        // Returns: "Space", "A" (Xbox), "Cross" (PlayStation)
    }
    
    // Save bindings
    private void SaveBindings()
    {
        string json = _input.asset.SaveBindingOverridesAsJson();
        PlayerPrefs.SetString("InputBindings", json);
    }
    
    // Load bindings
    private void LoadBindings()
    {
        string json = PlayerPrefs.GetString("InputBindings", string.Empty);
        if (!string.IsNullOrEmpty(json))
            _input.asset.LoadBindingOverridesFromJson(json);
    }
}
~~~

---

## Device Detection & Icon Switching

Detect which device the player is using to show correct button icons:

~~~csharp
private void OnEnable()
{
    _input.Player.Jump.performed += DetectDevice;
}

private void DetectDevice(InputAction.CallbackContext context)
{
    var device = context.control.device;
    
    if (device is Keyboard || device is Mouse)
    {
        ShowKeyboardIcons();
    }
    else if (device is Gamepad gamepad)
    {
        ShowGamepadIcons();
    }
}
~~~

Or listen globally:

~~~csharp
private void Awake()
{
    InputSystem.onActionChange += (obj, change) =>
    {
        if (change == InputActionChange.ActionPerformed)
        {
            var action = (InputAction)obj;
            var device = action.activeControl.device;
            UpdateButtonPrompts(device);
        }
    };
}
~~~

---

## UI Integration

Replace `StandaloneInputModule` with `InputSystemUIInputModule`:

1. Select EventSystem GameObject
2. Remove Standalone Input Module component
3. Add Component → Input System UI Input Module
4. Assign your `.inputactions` asset

UI navigation (Navigate, Submit, Cancel actions) will work automatically.

---

# Common Issues

| Problem                          | Cause                          | Fix                                      |
| -------------------------------- | ------------------------------ | ---------------------------------------- |
| Input not working                | Action map not enabled         | Call `_input.Player.Enable()`            |
| Null reference on `.current`     | Device not connected           | Check `if (Keyboard.current != null)`    |
| Input works in editor, not build | Wrong Active Input Handling    | Project Settings → Input System Package  |
| Callbacks fire multiple times    | Multiple subscriptions         | Unsubscribe in `OnDisable`               |
| Can't click UI                   | Wrong input module             | Use `InputSystemUIInputModule`           |
| Action not found in generated class | Didn't regenerate after changes | Click "Apply" after editing actions   |
| Method not called (PlayerInput)  | Wrong method name              | Must match "On" + ActionName exactly     |

---

# Quick Reference

## Generated Class Approach

~~~csharp
// Setup
private PlayerControls _input;

private void Awake()
{
    _input = new PlayerControls();
}

private void OnEnable()
{
    _input.Player.Enable();
    _input.Player.Jump.performed += OnJump;
}

private void OnDisable()
{
    _input.Player.Jump.performed -= OnJump;
    _input.Player.Disable();
}

// Reading values
Vector2 move = _input.Player.Move.ReadValue<Vector2>();
bool pressed = _input.Player.Jump.IsPressed();
bool justPressed = _input.Player.Jump.WasPressedThisFrame();
bool justReleased = _input.Player.Jump.WasReleasedThisFrame();

// Switching action maps
_input.Player.Disable();
_input.UI.Enable();
~~~

## PlayerInput Component Approach

~~~csharp
// Add PlayerInput component in Inspector
// Set Behavior to "Send Messages"

// Methods auto-called by Unity
public void OnMove(InputValue value)
{
    Vector2 input = value.Get<Vector2>();
}

public void OnJump(InputValue value)
{
    if (value.isPressed)
        Jump();
}

// Switching action maps
GetComponent<PlayerInput>().SwitchCurrentActionMap("UI");
~~~

---

# Godot Comparison

| Godot                            | Unity Input System                  |
| -------------------------------- | ----------------------------------- |
| Input Map (Project Settings)     | Input Actions Asset                 |
| `Input.is_action_pressed()`      | `action.IsPressed()`                |
| `Input.is_action_just_pressed()` | `action.WasPressedThisFrame()`      |
| `Input.get_vector()`             | `action.ReadValue<Vector2>()`       |
| Action name strings              | Generated C# class with type safety |
| Built-in                         | Requires package install            |

---

# Recommendations

**Use Generated C# Class if:**
- You're comfortable with code
- You want type safety and refactoring support
- You need precise control over when actions are enabled

**Use PlayerInput Component if:**
- You want minimal code
- You're prototyping quickly
- Designers need to configure input in Inspector

**Start with Generated Class.** It's the more flexible approach and scales better to larger projects.

---

# Related

- [[Communication Patterns in Unity]] — Event-driven input handling
- [[Reference Objects in Unity]] — Accessing InputManager as a service
- [[Scene Management in Unity]] — Persistent input managers across scenes
