The godot engine's way to enable rich text with animations and effects right in the engine with tags

## Setup

Enable BBCode by setting `bbcode_enabled = true` on your RichTextLabel node.
Set the minimum size if in the special containers

## Text Formatting

| Tag                 | Effect        | Example              |
| ------------------- | ------------- | -------------------- |
| `[b]text[/b]`       | Bold          | `[b]bold[/b]`        |
| `[i]text[/i]`       | Italic        | `[i]italic[/i]`      |
| `[u]text[/u]`       | Underline     | `[u]underline[/u]`   |
| `[s]text[/s]`       | Strikethrough | `[s]crossed out[/s]` |
| `[code]text[/code]` | Monospace     | `[code]code[/code]`  |

## Colors

| Tag                             | Effect                               |
| ------------------------------- | ------------------------------------ |
| `[color=name]text[/color]`      | Named color (red, green, blue, etc.) |
| `[color=#RRGGBB]text[/color]`   | Hex color (6 digits)                 |
| `[color=#RRGGBBAA]text[/color]` | Hex with alpha (8 digits)            |
| `[bgcolor=color]text[/bgcolor]` | Background color                     |
| `[fgcolor=color]text[/fgcolor]` | Foreground color (for "redacting")   |

## Layout & Alignment

| Tag                     | Effect                       |
| ----------------------- | ---------------------------- |
| `[center]text[/center]` | Center align                 |
| `[left]text[/left]`     | Left align                   |
| `[right]text[/right]`   | Right align                  |
| `[fill]text[/fill]`     | Justify (fill width)         |
| `[p]text[/p]`           | Paragraph (supports options) |
| `[indent]text[/indent]` | Indent text                  |
| `[br]`                  | Line break                   |
| `[hr]`                  | Horizontal rule              |

## Links & Tooltips

|Tag|Effect|
|---|---|
|`[url]link[/url]`|Display clickable link|
|`[url=link]text[/url]`|Clickable text|
|`[hint="tooltip"]text[/hint]`|Hover tooltip|

**Note:** URL clicks require connecting to the `meta_clicked` signal.

## Images

```
[img]path[/img]
[img=width]path[/img]
[img=widthxheight]path[/img]
[img=valign]path[/img]
[img color=#ff0000 width=32 height=32]path[/img]
```

## Fonts

|Tag|Effect|
|---|---|
|`[font=path]text[/font]`|Custom font|
|`[font_size=size]text[/font_size]`|Font size|
|`[outline_size=size]text[/outline_size]`|Outline size|
|`[outline_color=color]text[/outline_color]`|Outline color|

## Lists

**Unordered:**

```
[ul]
item 1
item 2
[/ul]
```

**Ordered:**

```
[ol type=1]
item 1
item 2
[/ol]
```

Types: `1` (numbers), `a`/`A` (letters), `i`/`I` (Roman numerals)

## Tables

```
[table=columns]
[cell]content[/cell]
[cell]content[/cell]
[/table]
```

Example:

```
[table=2]
[cell]A[/cell][cell]B[/cell]
[cell]C[/cell][cell]D[/cell]
[/table]
```

## Special Characters

|Tag|Character|
|---|---|
|`[lb]`|`[` (left bracket)|
|`[rb]`|`]` (right bracket)|
|`[char=codepoint]`|Unicode character (hex)|

## Animated Effects

| Effect  | Syntax                                                   |
| ------- | -------------------------------------------------------- |
| Pulse   | `[pulse freq=1.0 color=#ffffff40 ease=-2.0]text[/pulse]` |
| Wave    | `[wave amp=50.0 freq=5.0]text[/wave]`                    |
| Tornado | `[tornado radius=10.0 freq=1.0]text[/tornado]`           |
| Shake   | `[shake rate=20.0 level=5]text[/shake]`                  |
| Fade    | `[fade start=4 length=14]text[/fade]`                    |
| Rainbow | `[rainbow freq=1.0 sat=0.8 val=0.8]text[/rainbow]`       |

## Critical Notes

- Tags don't nest arbitrarily: `[b]bold[i]both[/b]still italic[/i]` won't work. Use: `[b]bold[i]both[/i][/b][i]still italic[/i]`
- Bold/italic look better with custom fonts loaded in theme overrides
- `[code]` requires a custom monospace font to work properly
- Leading/trailing whitespace is preserved (unlike HTML)

## Escaping User Input

```gdscript
func escape_bbcode(text: String) -> String:
    return text.replace("[", "[lb]")
```

This prevents BBCode injection in user-generated content.