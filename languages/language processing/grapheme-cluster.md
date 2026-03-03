# Grapheme Cluster

The smallest unit a user perceives as a single character, which may be composed of multiple Unicode code points.

## What it does

A grapheme cluster is the "visible unit" of text — a letter, emoji, or flag that appears as one character on screen. It can be composed of one or more Unicode code points. A single grapheme cluster may span multiple bytes in UTF-8 and multiple code points.

| Visible character | Unicode code points   | UTF-8 bytes | Notes                       |
| ----------------- | --------------------- | ----------- | --------------------------- |
| `a`               | `U+0061`              | 1           | Basic Latin                 |
| `😊`              | `U+1F60A`             | 4           | Emoji                       |
| `á`               | `U+0061 + U+0301`     | 3           | Base letter + combining mark|
| `👨‍👩‍👧‍👦`         | `U+1F468 + ZWJ + ...` | many        | ZWJ sequence                |

**1 grapheme cluster ≠ 1 Unicode code point ≠ 1 byte**

Required for correct cursor movement and string length measurement visible to users.

## Sources

- https://www.unicode.org/reports/tr29/#Grapheme_Cluster_Boundaries

## Related

- [[unicode]]
- [[UTF-8]]
- [[Rope]]
- [[parser]]

## Process

- Why does splitting a string by byte index rather than grapheme cluster boundary corrupt text?
- How does a zero-width joiner (ZWJ) compose multiple code points into a single visible character?
- What does "grapheme cluster" have to do with cursor movement in a text editor?
