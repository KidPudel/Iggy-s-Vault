# Unity Layout Components

## HorizontalLayoutGroup / VerticalLayoutGroup

| Property | Type | Description |
|---|---|---|
| `padding` | `RectOffset` | left, right, top, bottom |
| `spacing` | `float` | Space between children |
| `childAlignment` | `TextAnchor` | UpperLeft through LowerRight |
| `childControlWidth/Height` | `bool` | Layout controls child size |
| `childScaleWidth/Height` | `bool` | Factor in child scale |
| `childForceExpandWidth/Height` | `bool` | Force expand |
| `reverseArrangement` | `bool` | Reverse child order |

## GridLayoutGroup

| Property | Type | Description |
|---|---|---|
| `cellSize` | `Vector2` | Size of each cell |
| `spacing` | `Vector2` | Space between cells |
| `startCorner` | `Corner` | UpperLeft, UpperRight, LowerLeft, LowerRight |
| `startAxis` | `Axis` | Horizontal or Vertical |
| `constraint` | `Constraint` | Flexible, FixedColumnCount, FixedRowCount |
| `constraintCount` | `int` | Number of rows/columns |

## LayoutElement

| Property | Type | Description |
|---|---|---|
| `ignoreLayout` | `bool` | Exclude from layout |
| `minWidth/Height` | `float` | Minimum size |
| `preferredWidth/Height` | `float` | Preferred size |
| `flexibleWidth/Height` | `float` | Flexible weight |
| `layoutPriority` | `int` | Priority |

Sizing order: `min` -> `preferred` -> `flexible`

## ContentSizeFitter

| Property | Type |
|---|---|
| `horizontalFit` | `FitMode` (Unconstrained, MinSize, PreferredSize) |
| `verticalFit` | `FitMode` (Unconstrained, MinSize, PreferredSize) |

## Sources
- [Unity Manual: Auto Layout](https://docs.unity3d.com/Manual/UIAutoLayout.html)
