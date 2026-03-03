# Unity UI Toolkit

## UXML Elements

```xml
<ui:UXML xmlns:ui="UnityEngine.UIElements">
    <ui:VisualElement name="root" class="container">
        <ui:Label text="Hello" />
        <ui:Button text="Click" name="btn" />
        <ui:TextField label="Name" value="default" />
        <ui:Toggle label="Enable" value="true" />
        <ui:Slider label="Amount" low-value="0" high-value="1" />
        <ui:SliderInt label="Count" low-value="0" high-value="100" />
        <ui:DropdownField label="Choice" choices="A,B,C" />
        <ui:Foldout text="Details"><ui:Label text="Content" /></ui:Foldout>
        <ui:ScrollView />
        <ui:ListView name="list" />
        <ui:ProgressBar title="Loading" value="75" />
        <ui:RadioButtonGroup label="Options" choices="X,Y,Z" />
    </ui:VisualElement>
</ui:UXML>
```

## USS (Styling)

```css
Label { font-size: 14px; color: white; }
.header { font-size: 24px; -unity-font-style: bold; }
#my-button { width: 200px; height: 40px; background-color: rgb(60, 60, 60); }
Button:hover { background-color: rgb(80, 80, 80); }
.container > Label { margin: 5px; }
```

Key properties: `width/height`, `margin-*`, `padding-*`, `flex-grow/shrink/basis`, `flex-direction`, `justify-content`, `align-items`, `position` (relative/absolute), `background-color`, `border-*`, `color`, `font-size`, `-unity-font-style`, `-unity-text-align`, `opacity`, `display` (flex/none), `visibility`, `transition-property/duration`.

## C# API

```csharp
using UnityEngine.UIElements;

VisualElement root = uiDocument.rootVisualElement;

// Query
Button btn = root.Q<Button>("my-button");
VisualElement el = root.Q(className: "header");
root.Query<Label>().ForEach(l => l.text = "updated");

// Events
btn.clicked += OnClicked;
btn.RegisterCallback<ClickEvent>(OnClick);

// Style
btn.style.backgroundColor = new Color(0.2f, 0.2f, 0.2f);
btn.style.display = DisplayStyle.None;

// Classes
btn.AddToClassList("active");
btn.RemoveFromClassList("active");
btn.ToggleInClassList("active");

// DOM
root.Add(new Label("Dynamic"));
root.Insert(0, element);
root.Remove(element);
root.Clear();
```

## Common Events

| Event | Description |
|---|---|
| `ClickEvent` | Clicked |
| `MouseEnterEvent` / `MouseLeaveEvent` | Mouse hover |
| `PointerEnterEvent` / `PointerLeaveEvent` | Pointer (includes touch) |
| `FocusInEvent` / `FocusOutEvent` | Focus |
| `KeyDownEvent` / `KeyUpEvent` | Keyboard |
| `ChangeEvent<T>` | Value changed |
| `GeometryChangedEvent` | Resized |
| `NavigationMoveEvent` | Arrow keys / gamepad |
| `TransitionEndEvent` | USS transition done |

```csharp
textField.RegisterValueChangedCallback(evt => { /* evt.previousValue, evt.newValue */ });
```

## Sources
- [Unity Manual: UI Toolkit](https://docs.unity3d.com/Manual/UIElements.html)
