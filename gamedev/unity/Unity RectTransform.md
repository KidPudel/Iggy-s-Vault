# Unity RectTransform

Extends `Transform` for 2D UI layout. All UI elements have a RectTransform instead of Transform.

## Properties

| Property | Type | Description |
|---|---|---|
| `anchorMin` | `Vector2` | Lower-left anchor (0,0 to 1,1 normalized) |
| `anchorMax` | `Vector2` | Upper-right anchor (0,0 to 1,1 normalized) |
| `anchoredPosition` | `Vector2` | Position relative to anchor |
| `sizeDelta` | `Vector2` | Size relative to anchor spread |
| `pivot` | `Vector2` | Pivot point (0,0 to 1,1) |
| `rect` | `Rect` | Calculated rectangle (read only) |
| `offsetMin` | `Vector2` | Lower-left corner offset from anchor |
| `offsetMax` | `Vector2` | Upper-right corner offset from anchor |

## Common Anchor Presets

| Preset | anchorMin | anchorMax |
|---|---|---|
| Top-Left | (0, 1) | (0, 1) |
| Center | (0.5, 0.5) | (0.5, 0.5) |
| Bottom-Right | (1, 0) | (1, 0) |
| Stretch Horizontal | (0, 0.5) | (1, 0.5) |
| Stretch Vertical | (0.5, 0) | (0.5, 1) |
| Stretch Both | (0, 0) | (1, 1) |

> [!info]
> When `anchorMin == anchorMax`, `sizeDelta` is the element's absolute size. When anchors differ, `sizeDelta` is the size difference from the anchor-defined rect. Negative `sizeDelta` shrinks inward.

## Methods

| Method | Signature |
|---|---|
| `SetSizeWithCurrentAnchors` | `void SetSizeWithCurrentAnchors(Axis axis, float size)` |
| `SetInsetAndSizeFromParentEdge` | `void SetInsetAndSizeFromParentEdge(Edge edge, float inset, float size)` |
| `GetWorldCorners` | `void GetWorldCorners(Vector3[] fourCornersArray)` |
| `ForceUpdateRectTransforms` | `void ForceUpdateRectTransforms()` |

## Sources
- [Unity Scripting API: RectTransform](https://docs.unity3d.com/ScriptReference/RectTransform.html)
