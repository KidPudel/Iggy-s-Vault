# Unity UI Components

## Button (`UnityEngine.UI.Button`)

| Property/Event | Type | Description |
|---|---|---|
| `onClick` | `Button.ButtonClickedEvent` | Fires on click |
| `interactable` | `bool` | Can be clicked |
| `transition` | `Transition` | None, ColorTint, SpriteSwap, Animation |
| `colors` | `ColorBlock` | Normal, Highlighted, Pressed, Selected, Disabled colors |

```csharp
button.onClick.AddListener(() => Debug.Log("Clicked"));
button.onClick.RemoveAllListeners();
```

## TextMeshPro (`TMPro.TextMeshProUGUI`)

`using TMPro;`

| Property | Type | Description |
|---|---|---|
| `text` | `string` | Text content (supports rich text) |
| `fontSize` | `float` | Font size |
| `font` | `TMP_FontAsset` | Font asset |
| `color` | `Color` | Text color |
| `alignment` | `TextAlignmentOptions` | Text alignment |
| `enableAutoSizing` | `bool` | Auto-fit to container |
| `overflowMode` | `TextOverflowModes` | Overflow, Ellipsis, Truncate, etc. |
| `maxVisibleCharacters` | `int` | For typewriter effect |

Rich text: `<b>bold</b>`, `<i>italic</i>`, `<size=20>`, `<color=#FF0000>`, `<sprite index=0>`, `<link="id">clickable</link>`

## Image (`UnityEngine.UI.Image`)

| Property | Type | Description |
|---|---|---|
| `sprite` | `Sprite` | Source sprite |
| `color` | `Color` | Tint color |
| `type` | `Image.Type` | Simple, Sliced, Tiled, Filled |
| `fillMethod` | `Image.FillMethod` | Horizontal, Vertical, Radial90/180/360 |
| `fillAmount` | `float` | 0-1 |
| `preserveAspect` | `bool` | Maintain aspect ratio |
| `raycastTarget` | `bool` | Receives raycasts |

## ScrollRect (`UnityEngine.UI.ScrollRect`)

| Property | Type | Description |
|---|---|---|
| `content` | `RectTransform` | Scrollable content |
| `horizontal` / `vertical` | `bool` | Allow scrolling |
| `movementType` | `MovementType` | Unrestricted, Elastic, Clamped |
| `inertia` | `bool` | Continue after release |
| `normalizedPosition` | `Vector2` | Scroll position (0-1) |
| `onValueChanged` | `ScrollRectEvent` | Fires on scroll |

## Slider (`UnityEngine.UI.Slider`)

| Property | Type | Description |
|---|---|---|
| `value` | `float` | Current value |
| `minValue` / `maxValue` | `float` | Range |
| `wholeNumbers` | `bool` | Integer only |
| `onValueChanged` | `SliderEvent` | Fires on change |

## Toggle (`UnityEngine.UI.Toggle`)

| Property | Type | Description |
|---|---|---|
| `isOn` | `bool` | Current state |
| `group` | `ToggleGroup` | For radio-button behavior |
| `onValueChanged` | `ToggleEvent` | Fires on change (bool) |

## TMP_Dropdown (`TMPro.TMP_Dropdown`)

| Property | Type | Description |
|---|---|---|
| `value` | `int` | Selected index |
| `options` | `List<OptionData>` | Options list |
| `onValueChanged` | `DropdownEvent` | Fires on selection (int) |

```csharp
dropdown.ClearOptions();
dropdown.AddOptions(new List<string> { "A", "B", "C" });
```

## TMP_InputField (`TMPro.TMP_InputField`)

| Property | Type | Description |
|---|---|---|
| `text` | `string` | Current text |
| `contentType` | `ContentType` | Standard, IntegerNumber, Password, etc. |
| `characterLimit` | `int` | Max chars (0 = unlimited) |
| `onValueChanged` | `OnChangeEvent` | Each character change |
| `onEndEdit` | `SubmitEvent` | Editing ends |
| `onSubmit` | `SubmitEvent` | Enter key |

## Mask vs RectMask2D

| Component | Description |
|---|---|
| `Mask` | Uses Image alpha + stencil buffer |
| `RectMask2D` | Clips to RectTransform bounds, no stencil, more performant |

## Sources
- [Unity Manual: Unity UI](https://docs.unity3d.com/Manual/UIToolkits.html)
- [TextMeshPro](https://docs.unity3d.com/Packages/com.unity.textmeshpro@3.0/manual/index.html)
