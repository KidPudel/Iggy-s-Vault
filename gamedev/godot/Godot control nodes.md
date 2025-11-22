# Godot Control Nodes Reference

https://www.youtube.com/watch?v=5Hog6a0EYa0

## Core Concept

Control nodes are the foundation of Godot's UI system. They have green icons in the editor and unique properties that make them work well together. Unlike Node2D/Sprite nodes, Control nodes understand anchoring, margins, and container relationships.

---

## Essential Display Nodes

### Label

**Purpose:** Display text  
**Key Properties:**

- `text` - The displayed text
- `autowrap_mode` - Word wrapping behavior
- `horizontal_alignment` / `vertical_alignment` - Text alignment

**When to use:** Any static text display - health values, menu items, descriptions

**Gotcha:** Can't be clicked by default (unlike Button). If you need clickable text, use Button instead.

---

### TextureRect

**Purpose:** Display images/textures in UI  
**Key Properties:**

- `texture` - The image to display
- `expand_mode` - How texture scales (Keep, Keep Centered, Ignore Size, Keep Aspect, etc.)
- `stretch_mode` - Scale behavior (Scale, Tile, Keep, Keep Centered, Keep Aspect, etc.)

**When to use:**

- UI backgrounds (images)
- Icons
- Character portraits
- Any static image in your interface

**Difference from Sprite:** TextureRect is for UI with multiple scaling modes and container support. Sprite is for game world objects.

**Gotcha:** The texture won't scale by default - you need to set `expand_mode` to something other than "Keep Size" and configure `stretch_mode` appropriately.

---

### Panel

**Purpose:** Draw a styled *background* rectangle (purely visual)  
**Key Properties:**

- Theme overrides for stylebox appearance

**When to use:** When you want a decorative background that doesn't affect layout

**Difference from TextureRect**: Panel is meant for UI chrome/decoration while TextureRect is meant for actual image content. You'll often use them together - Panel as the container background (stylebox), TextureRect for images inside it.

**Critical distinction:** Panel does NOTHING to its children. It's just decoration. Children position themselves independently.

---

### PanelContainer

**Purpose:** Draw styled background AND manage children's layout  
**Key Properties:**

- Theme overrides for stylebox appearance
- Automatically sizes to fit children (with padding/margins from the stylebox)

**When to use:** When you want a styled box that also handles spacing and sizing of its contents

**Why it's confusing:** The name "Panel" sounds like it should be the container one, but it's not. If you want container behavior, you need PanelContainer.

**Example use case:** A dialog box where you want the background panel to grow/shrink with its content and respect padding.

---

### Button

**Purpose:** Clickable button with text  
**Key Properties:**

- `text` - Button label
- `disabled` - Prevent clicking
- `toggle_mode` - Makes it a toggle button (stays pressed)

**Signals:**

- `pressed()` - When clicked
- `toggled(toggled_on)` - When toggle state changes

**When to use:** Any interactive button

**Gotcha:** Button only holds single-line text. For multiline, you'll need custom solutions (like putting a Label/RichTextLabel as a child with mouse_filter set to Ignore).

---

### TextureButton

**Purpose:** Button with custom textures for different states  
**Key Properties:**

- `texture_normal` - Default appearance
- `texture_pressed` - When clicked
- `texture_hover` - When mouse hovers
- `texture_disabled` - When disabled
- `texture_focused` - When keyboard focused

**When to use:** Custom-styled buttons with image assets

**Gotcha:** All five texture states are independent - you need to provide each one or the button might look weird in different states.

---

### LineEdit

**Purpose:** Single-line text input  
**Key Properties:**

- `text` - Current text value
- `placeholder_text` - Hint text when empty
- `max_length` - Character limit
- `secret` - Hide text (password fields)

**Signals:**

- `text_changed(new_text)` - As user types
- `text_submitted(text)` - When user presses Enter

**When to use:** Username fields, search bars, any single-line input

---

### RichTextLabel

**Purpose:** Display formatted text with BBCode markup  
**Key Properties:**

- `bbcode_enabled` - Enable BBCode formatting
- `text` - The raw text
- `bbcode_text` - Text with formatting tags
- `fit_content_height` - Auto-resize height to fit content
- `scroll_active` - Enable scrolling

**BBCode examples:**

- `[b]bold[/b]`
- `[i]italic[/i]`
- `[color=red]colored[/color]`
- `[url]https://example.com[/url]`

**When to use:** Dialogue boxes, formatted game text, tooltips with styling, text logs

**Gotcha:** Has built-in scrolling but it's tricky - consider using ScrollContainer instead for more control.

---

## Essential Container Nodes

**CRITICAL RULE:** When a Control node is a child of a Container, the container controls its size and position. You cannot manually set anchors, margins, or rect_size on container children - the container overwrites them.

Container nodes care about children sizes and adjust itself.
**Containers** (VBoxContainer, HBoxContainer, GridContainer, PanelContainer, MarginContainer, etc.):

- Size themselves based on children
- Control children's positions and sizes
- React when children change

**Non-containers** (Panel, NinePatchRect, TextureRect, Control, Label, Button, etc.):

- Ignore children's sizes completely
- Children can be any size, anywhere
- You manually position/size everything

---

### MarginContainer

**Purpose:** Add padding/margins around a single child  
**Key Properties:**

- `theme_override_constants/margin_left`, `margin_right`, `margin_top`, `margin_bottom` - Padding for each side (default: 0)

**When to use:**

- Adding space between UI edge and content
- Creating breathing room around elements

**Example:** Put a lifebar in a MarginContainer to keep it away from screen edges.

---

### VBoxContainer

**Purpose:** Arrange children vertically in a column  
**Key Properties:**

- `theme_override_constants/separation` - Spacing between children
- `alignment` - How to align children (begin, center, end)

**When to use:** Vertical menus, lists, stacked UI elements

**How children size works:** By default, each child takes its minimum size. Use `size_flags_vertical` on children to control expansion:

- `SIZE_FILL` - Take available space
- `SIZE_EXPAND` + `SIZE_FILL` - Expand to fill available space

---

### HBoxContainer

**Purpose:** Arrange children horizontally in a row  
**Key Properties:**

- `theme_override_constants/separation` - Spacing between children
- `alignment` - How to align children (begin, center, end)

**When to use:** Horizontal button rows, health bar with icon, any side-by-side layout

**Same sizing rules as VBoxContainer** but horizontally.

---

### GridContainer

**Purpose:** Arrange children in a grid pattern  
**Key Properties:**

- `columns` - Number of columns (rows calculated automatically)
- `theme_override_constants/h_separation` - Horizontal spacing
- `theme_override_constants/v_separation` - Vertical spacing

**When to use:**

- Inventory grids
- Skill trees
- Any grid-based UI

**How it works:** You only set columns. If you have 9 children and set `columns = 3`, you automatically get 3 rows (9÷3=3). Add more children and rows increase automatically.

**Gotcha:** If a column or row has no child with a minimum size, it can squish to zero width/height.

---

### CenterContainer

**Purpose:** Keep all children centered  
**Key Properties:**

- `use_top_left` - Center relative to top-left corner instead of true center

**When to use:**

- Centering a single element (like a menu)
- Title screens
- Modal dialogs

**Gotcha:** If your children have no minimum size, they'll squish to zero and you won't see anything.

---

### ScrollContainer

**Purpose:** Add scrolling to content that overflows  
**Key Properties:**

- `horizontal_scroll_mode` - Show/hide/auto horizontal scrollbar
- `vertical_scroll_mode` - Show/hide/auto vertical scrollbar
- `follow_focus` - Auto-scroll to keep focused child visible

**When to use:**

- Long text logs
- Item lists that exceed screen space
- Any scrollable area

**How it works:** Put content inside (usually a VBoxContainer with many children), and scrollbars appear when content exceeds the container's size.

**Gotcha:** Only works properly with ONE direct child. Put multiple items in a VBoxContainer/HBoxContainer first, then put that container in the ScrollContainer.

---

### AspectRatioContainer

**Purpose:** Maintain aspect ratio of children when resizing  
**Key Properties:**

- `ratio` - The aspect ratio to maintain (width/height)
- `stretch_mode` - How to fit (fit, cover, etc.)
- `alignment_horizontal` / `alignment_vertical` - Where to position child

**When to use:**

- Game viewport in UI
- Video players
- Any content that must maintain specific proportions

---

## Size Flags (For Children in Containers)

Every Control node has size flags that tell containers how to treat it:

### Horizontal/Vertical Flags:

- **FILL** - Take up available space (but respect siblings)
- **EXPAND** - Request more space (will expand to fill)
- **SHRINK_CENTER** / **SHRINK_BEGIN** / **SHRINK_END** - Shrink if possible, align accordingly

**Expand vs Fill:**

- `FILL` alone = "I'll take space but only my share"
- `EXPAND + FILL` = "Give me as much space as you can"

### Stretch Ratio:

Determines how much space expanded controls take relative to each other.

- If 3 children all have ratio=1, each gets 1/3 of space
- If first child has ratio=2, others have ratio=1: first gets 2/4 (50%), others get 1/4 (25%) each

---

## Common Patterns

### Simple Menu

```
VBoxContainer
├── Button ("Start Game")
├── Button ("Options")
└── Button ("Quit")
```

### Health Bar with Icon

```
HBoxContainer
├── TextureRect (heart icon)
└── TextureProgress (health bar)
```

### Inventory Grid with Padding

```
MarginContainer
└── GridContainer (columns=5)
    ├── Panel (slot 1)
    ├── Panel (slot 2)
    └── ... (more slots)
```

### Dialog Box

```
PanelContainer
└── VBoxContainer
    ├── Label (title)
    ├── RichTextLabel (message)
    └── HBoxContainer
        ├── Button ("OK")
        └── Button ("Cancel")
```

### Scrollable Text Log

```
ScrollContainer
└── VBoxContainer
    ├── Label (entry 1)
    ├── Label (entry 2)
    └── ... (more entries added via code)
```

---

## Layout Workflow

1. **Start with a root Control** that covers the entire screen (full rect) - this does nothing except give you a base
2. **Anchor your top-level UI** to screen corners/edges using Layout presets (Top Left, Top Right, etc.)
3. **Use containers** to arrange children - don't manually position things inside containers
4. **Set size flags** on container children to control expansion behavior
5. **Use MarginContainer** to add padding/spacing when needed

**Golden rule:** Let containers do the positioning. If you're manually adjusting margins and positions constantly, you're fighting the system.

---

## Common Mistakes

1. **Using Panel when you want PanelContainer** - Panel doesn't manage children
2. **Manually positioning nodes inside containers** - The container will override you
3. **Forgetting to set size flags** - Children won't expand as expected
4. **Not setting minimum sizes** - Elements can shrink to zero in some containers
5. **Putting multiple direct children in ScrollContainer** - Use a single VBox/HBox container to hold multiple items

---

## Anchors & Margins (For Non-Container Children)

When a Control is NOT in a container, you position it with anchors and margins:

**Anchors** (0.0 to 1.0):

- Define attachment points on the parent
- `(0, 0)` = top-left
- `(1, 1)` = bottom-right
- `(0.5, 0.5)` = center

**Margins/Offsets:**

- Distance from anchor points
- Negative values pull toward anchor

**Layout Presets:** Use the "Layout" dropdown in the editor - it sets both anchors and offsets for common cases (Full Rect, Top Left, Center, etc.)

**When in doubt:** Use Layout presets and containers. Manual anchor/margin math is where most beginners get lost.

---

## Pro Tips

- **Start with containers, add manual positioning only when necessary**
- **Use ColorRect (temporarily) as a child set to Full Rect to visualize container bounds while designing**
- **Check "Expand Non-Default" in the inspector to see what properties you've changed**
- **Nested containers are normal** - VBox inside HBox inside MarginContainer is common and good
- **The editor's Layout menu is your friend** - use it instead of manually typing anchor values

This reference covers 90% of what you'll use for game UIs. Master these nodes and you'll rarely get stuck.