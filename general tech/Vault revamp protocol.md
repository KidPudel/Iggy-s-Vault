# Vault Revamp Protocol

Purpose: keep Obsidian as a flat atomic wiki, and keep deep learning in spatial tools (Apple Freeform/paper).

## Core Rule

- Obsidian note = one factor/concept only.
- Spatial board = relationships, systems, tradeoffs, dynamic understanding.

If a note tries to teach a full lesson, model many interacting factors, or replace thinking, it should be split or moved to spatial work.

## What Belongs in Obsidian

- Terminology and definitions.
- API behavior and syntax facts.
- Config/setup commands.
- Short cheat-sheets and lookup tables.
- Small index notes (link hubs only).
- Interview prep banks/checklists (explicitly allowed as retrieval-oriented wiki notes).

## What Does Not Belong as a Single Obsidian Note

- Learning paths and roadmaps.
- "Complete foundation" tutorials.
- Design/theme/system understanding docs that require relationship modeling.

These should live in Freeform/paper. Obsidian should only hold distilled outputs and atomic facts.

## Note Types

- `concept`: single definition + why it matters + links.
- `reference`: lookup for stable facts.
- `cheat-sheet`: terse command/formula list.
- `index`: links only; no long explanations.
- `spatial-bridge`: pointer to Freeform/paper map for a theme.

## Atomic Note Template

```md
# <Concept Name>

## Definition
<single clear definition>

## Why it matters
<2-4 bullets>

## Boundaries
- Not:
- Related but different:

## See also
- [[Related Note A]]
- [[Related Note B]]
```

## Conversion Rules

1. Keep filename stable when possible (to preserve backlinks).
2. Turn overloaded note into an `index` or `spatial-bridge`.
3. Extract sections into new atomic notes.
4. Keep each atomic note under one concept.
5. Remove tutorial voice; keep factual/reference voice.

## Quick Quality Checks

- Can title be answered in one sentence?
- Does the note define one thing only?
- Could a section become its own note? If yes, split.
- Is this understanding/model-building content? If yes, move to spatial board.
