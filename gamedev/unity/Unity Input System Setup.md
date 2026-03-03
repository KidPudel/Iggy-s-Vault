# Unity Input System Setup

Installing, configuring, and defining Input Actions in the Unity Input System package.

## Sources
- [Unity Manual: Input System Installation](https://docs.unity3d.com/Packages/com.unity.inputsystem@1.7/manual/Installation.html)
- [Unity Manual: Input Action Assets](https://docs.unity3d.com/Packages/com.unity.inputsystem@1.7/manual/ActionAssets.html)
- [Unity Manual: Interactions](https://docs.unity3d.com/Packages/com.unity.inputsystem@1.7/manual/Interactions.html)
- [Unity API: InputSystemUIInputModule](https://docs.unity3d.com/Packages/com.unity.inputsystem@1.7/api/UnityEngine.InputSystem.UI.InputSystemUIInputModule.html)

## Related topics
- [[Unity Input System Implementation]]
- [[Input System in Unity]]
- [[MonoBehaviour Reference]]
- [[Unity Index Book of References]]

---

## Package Installation

1. **Window > Package Manager > Unity Registry** > search "Input System" > Install
2. Unity prompts to enable the new input backend -- select **Yes**
3. Unity restarts the editor

### Active Input Handling (Project Settings > Player)

| Setting | Effect |
|---|---|
| Input System Package (New) | Only new Input System active |
| Input Manager (Old) | Only legacy `Input.GetKey` etc. active |
| Both | Both systems active simultaneously |

> [!warning]
> Selecting "Input System Package (New)" disables all legacy `UnityEngine.Input` calls. Existing code using `Input.GetAxis` will throw errors.

## Input Action Asset (.inputactions)

Create: **Right-click in Project > Create > Input Actions**

### Asset Structure

```
Input Action Asset
├── Action Map        (group of related actions, e.g., "Player", "UI")
│   ├── Action        (a single logical input, e.g., "Move", "Jump")
│   │   ├── Binding   (physical input path, e.g., <Keyboard>/space)
│   │   ├── Binding   (alternative binding)
│   │   └── Composite Binding (multiple keys forming one value)
│   └── Action
└── Action Map
```

### Action Types

| Type | Return | Description |
|---|---|---|
| `Value` | Continuous value | Produces `started`, `performed`, `canceled`. Automatically tracks the most-actuated bound control (conflict resolution). |
| `Button` | `float` (0 or 1) | Produces `started`, `performed`, `canceled`. Default press threshold: 0.5. |
| `PassThrough` | Same as Value | No conflict resolution. Passes through input from all bound controls simultaneously. Does not produce `started`/`canceled` -- only `performed`. |

### Control Types (set per Action when Type = Value or PassThrough)

| Control Type | C# Read Type | Example |
|---|---|---|
| `Any` | depends on binding | auto-determined |
| `Analog` | `float` | Trigger axis |
| `Axis` | `float` | Single axis (-1 to 1) |
| `Bone` | `UnityEngine.InputSystem.XR.Bone` | XR bone |
| `Button` | `float` | Binary button |
| `Digital` | `int` | Integer value |
| `Double` | `double` | Double precision |
| `Dpad` | `Vector2` | D-pad |
| `Eyes` | `UnityEngine.InputSystem.XR.Eyes` | XR eyes |
| `Integer` | `int` | Integer value |
| `Quaternion` | `Quaternion` | 3D rotation |
| `Stick` | `Vector2` | Analog stick |
| `Touch` | `TouchState` | Touch data |
| `Vector2` | `Vector2` | 2D axis |
| `Vector3` | `Vector3` | 3D axis |

## Interactions

Set on a Binding or Action. Modify when `performed` fires.

| Interaction | Behavior |
|---|---|
| `Default` | `performed` on actuation, `canceled` on release |
| `Press` | Configurable: Press Only, Release Only, Press And Release. `performed` per mode. |
| `Hold` | `performed` after held for `duration` seconds. `canceled` if released early. Default duration: `InputSettings.defaultHoldTime` (0.4s). |
| `Tap` | `performed` if pressed and released within `duration` seconds. `canceled` if held too long. Default duration: `InputSettings.defaultTapTime` (0.2s). |
| `SlowTap` | `performed` on release if held >= `duration` seconds. `canceled` if released too early. |
| `MultiTap` | `performed` after `tapCount` taps within `tapDelay` between each. Default count: 2 (double-tap). |

### Interaction Parameters

```
Hold:      duration (float, seconds)
Tap:       duration (float, seconds), pressPoint (float)
SlowTap:   duration (float, seconds), pressPoint (float)
MultiTap:  tapCount (int), tapDelay (float), tapTime (float), pressPoint (float)
Press:     pressPoint (float), behavior (PressOnly/ReleaseOnly/PressAndRelease)
```

## Composite Bindings

Combine multiple controls into one value.

| Composite | Output | Parts |
|---|---|---|
| `1D Axis` | `float` (-1 to 1) | `negative`, `positive` |
| `2D Vector` | `Vector2` | `up`, `down`, `left`, `right` |
| `3D Vector` | `Vector3` | `up`, `down`, `left`, `right`, `forward`, `backward` |
| `One Modifier` | varies | `modifier`, `binding` (e.g., Ctrl+C) |
| `Two Modifiers` | varies | `modifier1`, `modifier2`, `binding` |
| `Button With One Modifier` | `float` | `modifier`, `button` |
| `Button With Two Modifiers` | `float` | `modifier1`, `modifier2`, `button` |

### 2D Vector Composite Modes

| Mode | Behavior |
|---|---|
| `Analog` | Raw float values from each part |
| `Digital` | Each part is 0 or 1, result normalized |
| `DigitalNormalized` | Same as Digital (default) |

## Common Binding Paths

### Keyboard
```
<Keyboard>/space
<Keyboard>/w
<Keyboard>/a
<Keyboard>/s
<Keyboard>/d
<Keyboard>/escape
<Keyboard>/leftShift
<Keyboard>/rightShift
<Keyboard>/leftCtrl
<Keyboard>/enter
<Keyboard>/tab
<Keyboard>/backquote
<Keyboard>/1        (number keys)
<Keyboard>/numpad1  (numpad keys)
<Keyboard>/f1       (function keys)
```

### Mouse
```
<Mouse>/leftButton
<Mouse>/rightButton
<Mouse>/middleButton
<Mouse>/delta          (Vector2 - mouse movement)
<Mouse>/position       (Vector2 - screen position)
<Mouse>/scroll         (Vector2 - scroll wheel)
```

### Gamepad
```
<Gamepad>/leftStick          (Vector2)
<Gamepad>/rightStick         (Vector2)
<Gamepad>/leftStick/x        (float)
<Gamepad>/leftStick/y        (float)
<Gamepad>/buttonSouth        (A / Cross)
<Gamepad>/buttonNorth        (Y / Triangle)
<Gamepad>/buttonEast         (B / Circle)
<Gamepad>/buttonWest         (X / Square)
<Gamepad>/leftShoulder       (LB / L1)
<Gamepad>/rightShoulder      (RB / R1)
<Gamepad>/leftTrigger        (LT / L2, float)
<Gamepad>/rightTrigger       (RT / R2, float)
<Gamepad>/start
<Gamepad>/select
<Gamepad>/dpad               (Vector2)
<Gamepad>/dpad/up
<Gamepad>/dpad/down
<Gamepad>/leftStickPress     (L3)
<Gamepad>/rightStickPress    (R3)
```

## PlayerInput Component

Add to a GameObject: **Add Component > PlayerInput**

| Property | Values |
|---|---|
| Actions | Reference to `.inputactions` asset |
| Default Map | Which Action Map is active on start |
| UI Input Module | Reference to `InputSystemUIInputModule` (for UI navigation) |
| Camera | Associated camera (for split-screen) |
| Behavior | `Send Messages`, `Broadcast Messages`, `Invoke Unity Events`, `Invoke CSharp Events` |

### Behavior Modes

| Mode | Mechanism |
|---|---|
| Send Messages | Calls `OnMove(InputValue)`, `OnJump(InputValue)`, etc. on the same GameObject via `SendMessage` |
| Broadcast Messages | Same as Send Messages but uses `BroadcastMessage` (includes children) |
| Invoke Unity Events | Exposes UnityEvent fields per action in the Inspector |
| Invoke CSharp Events | Exposes `onActionTriggered` C# event |

## InputSystemUIInputModule

Replaces the legacy `StandaloneInputModule` for UI navigation with the new Input System.

| Property | Description |
|---|---|
| Move | Action for UI navigation (Vector2) |
| Submit | Action for confirming selection |
| Cancel | Action for canceling/back |
| Point | Action for pointer position (Vector2) |
| Click | Action for pointer click |
| ScrollWheel | Action for scroll input |
| MiddleClick | Action for middle mouse |
| RightClick | Action for right click |
| TrackedDevicePosition | XR pointer position |
| TrackedDeviceOrientation | XR pointer rotation |

## Generate C# Class from Asset

In the `.inputactions` asset Inspector:
1. Check **Generate C# Class**
2. Set **C# Class File** path
3. Set **C# Class Name**
4. Set **C# Class Namespace** (optional)
5. Click **Apply**

This generates a class with typed accessors for all Action Maps and Actions.
