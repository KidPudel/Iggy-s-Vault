# Unity EventSystem

Required for any UI interaction. One per scene. Created automatically with a Canvas.

Namespace: `using UnityEngine.EventSystems;`

## Components

| Component | Description |
|---|---|
| `EventSystem` | Core manager. Tracks current selected object, dispatches events. |
| `StandaloneInputModule` | Processes mouse/keyboard/controller input (legacy Input Manager) |
| `InputSystemUIInputModule` | Processes input from new Input System |

## EventSystem Properties

| Property | Type | Description |
|---|---|---|
| `current` | `EventSystem` (static) | Currently active EventSystem |
| `currentSelectedGameObject` | `GameObject` | Currently selected UI object |
| `firstSelectedGameObject` | `GameObject` | Object selected on start |
| `SetSelectedGameObject` | `void SetSelectedGameObject(GameObject go)` | Programmatically select |
| `IsPointerOverGameObject` | `bool IsPointerOverGameObject()` | True if pointer over any UI |

> [!warning]
> `IsPointerOverGameObject()` does not work reliably with the new Input System in `Update()`. Call it in `LateUpdate()`.

## IPointerHandler Interfaces

| Interface | Method | Fires when |
|---|---|---|
| `IPointerEnterHandler` | `OnPointerEnter(PointerEventData)` | Pointer enters |
| `IPointerExitHandler` | `OnPointerExit(PointerEventData)` | Pointer exits |
| `IPointerDownHandler` | `OnPointerDown(PointerEventData)` | Pointer pressed |
| `IPointerUpHandler` | `OnPointerUp(PointerEventData)` | Pointer released |
| `IPointerClickHandler` | `OnPointerClick(PointerEventData)` | Click completed |
| `IDragHandler` | `OnDrag(PointerEventData)` | During drag |
| `IBeginDragHandler` | `OnBeginDrag(PointerEventData)` | Drag begins |
| `IEndDragHandler` | `OnEndDrag(PointerEventData)` | Drag ends |
| `IDropHandler` | `OnDrop(PointerEventData)` | Dropped on element |
| `IScrollHandler` | `OnScroll(PointerEventData)` | Scroll wheel |
| `ISelectHandler` | `OnSelect(BaseEventData)` | Selected |
| `IDeselectHandler` | `OnDeselect(BaseEventData)` | Deselected |
| `ISubmitHandler` | `OnSubmit(BaseEventData)` | Submit action |
| `ICancelHandler` | `OnCancel(BaseEventData)` | Cancel action |

## PointerEventData

| Property | Type | Description |
|---|---|---|
| `position` | `Vector2` | Screen position |
| `delta` | `Vector2` | Movement since last frame |
| `button` | `InputButton` | Left, Right, Middle |
| `clickCount` | `int` | Number of clicks |
| `pointerEnter` | `GameObject` | Object under pointer |
| `pointerDrag` | `GameObject` | Object being dragged |
| `scrollDelta` | `Vector2` | Scroll wheel delta |

## Sources
- [Unity Scripting API: EventSystem](https://docs.unity3d.com/ScriptReference/EventSystems.EventSystem.html)
