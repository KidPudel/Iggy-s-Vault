# Unity Canvas

Every UI element must be a child of a Canvas.

## Render Modes

| Mode | Description | Sorting |
|---|---|---|
| Screen Space - Overlay | Rendered on top of everything, no camera reference. Scales to screen. | By hierarchy order. |
| Screen Space - Camera | Rendered at a set distance from a specified camera. Allows 3D effects. | Sorting Layer + Order in Layer |
| World Space | Canvas exists in world space. Has a RectTransform with world position/size. | Same as other 3D objects (by depth) |

## Canvas Properties

| Property | Type | Description |
|---|---|---|
| `renderMode` | `RenderMode` | Overlay, ScreenSpaceCamera, WorldSpace |
| `worldCamera` | `Camera` | Camera for ScreenSpaceCamera mode |
| `planeDistance` | `float` | Distance from camera in ScreenSpaceCamera mode |
| `sortingLayerID` | `int` | Sorting layer |
| `sortingOrder` | `int` | Order within sorting layer |
| `pixelPerfect` | `bool` | Snap elements to pixel boundaries |
| `scaleFactor` | `float` | Scale multiplier for Overlay canvas |
| `overrideSorting` | `bool` | Override parent canvas sorting |

## Canvas Scaler

| UI Scale Mode | Description |
|---|---|
| Constant Pixel Size | UI stays same pixel size regardless of screen |
| Scale With Screen Size | UI scales with reference resolution |
| Constant Physical Size | UI stays same physical size (uses DPI) |

**Scale With Screen Size** properties:

| Property | Description |
|---|---|
| Reference Resolution | Design resolution (e.g., 1920x1080) |
| Screen Match Mode | Match Width Or Height, Expand, Shrink |
| Match | 0 = match width, 1 = match height, 0.5 = blend |

## CanvasGroup

| Property | Type | Description |
|---|---|---|
| `alpha` | `float` | 0-1, transparency of group |
| `interactable` | `bool` | Can children receive input |
| `blocksRaycasts` | `bool` | Do children block raycasts |
| `ignoreParentGroups` | `bool` | Ignore parent CanvasGroup settings |

```csharp
canvasGroup.alpha = 0f;
canvasGroup.interactable = false;
canvasGroup.blocksRaycasts = false;
```

## Sources
- [Unity Manual: Canvas](https://docs.unity3d.com/Manual/UICanvas.html)
- [Unity Manual: Canvas Group](https://docs.unity3d.com/ScriptReference/CanvasGroup.html)
