# Canvas Layers in Godot

## What It Does

Canvas Layer changes the **draw order** of UI and 2D elements independently of the scene tree hierarchy.

## The Core Problem It Solves

Normally, Godot draws nodes in tree order (top to bottom). If you have:

```
- Player
- Enemy
- UI (health bar)
```

The UI draws last, appearing on top. But what if you add a "Game Over" screen later in the tree? It might draw behind your UI. Canvas Layer breaks you free from this constraint.

## Key Properties

- **Layer**: Integer value. Higher numbers = drawn on top
- **Offset**: Shifts everything on this layer (useful for screen shake on UI)
- **Follow Viewport**: Makes the layer follow camera movement or stay fixed

## When to Use It

1. **Persistent UI** - Health bars, score counters that always render above gameplay
2. **Menus/Overlays** - Pause menus, dialog boxes that must appear over everything
3. **Parallax backgrounds** - Different scroll speeds (though ParallaxBackground exists for this)
4. **Screen effects** - Vignettes, screen-space overlays independent of game world

## Quick Example

```
- World (layer 0 - default)
  - Player
  - Enemies
- CanvasLayer (layer 1)
  - HUD
- CanvasLayer (layer 2)  
  - PauseMenu
```

Game objects render first, HUD always on top, pause menu above that. Scene tree order becomes irrelevant.

## The Essence

Canvas Layer is about **explicit rendering control**. Use it when "what's last in the tree draws on top" isn't sufficient for your needs.