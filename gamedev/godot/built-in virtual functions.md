# Godot Built-in Virtual Functions

`init() `Constructor called when object is created, before ready(). Use for initializing variables that don't depend on the scene tree.

`ready()` Called once when node enters scene tree. Children are guaranteed to exist here. Your main setup function.

`entertree()` Called before ready() when node enters scene tree. Children may not exist yet. Good for scene-independent initialization.

`exittree()` Called when node leaves scene tree. Cleanup connections and resources here.

`process(delta)` Called every frame. Delta is time since last frame in seconds. Use for frame-dependent logic like movement.

`physicsprocess(delta)` Called every physics frame (default 60 FPS, fixed timestep). Use for physics calculations and rigidbody movement.

`input(event)` Receives all unhandled input events. Runs before unhandledinput(). Good for UI and high-priority input.

`unhandledinput(event)` Receives input events that weren't consumed by UI or input(). Best for gameplay controls with exception for continuous movement, use process

`unhandledkeyinput(event)` Like unhandledinput() but only for keyboard events. Slightly faster if you only need keys.

`guiinput(event)`: scoped to that Control's, For Control nodes only, UI elements process input here, This is where UI "consumes" events

`notification(what)` Low-level callback receiving notifications (constants like NOTIFICATIONPAUSED). Use when other callbacks don't fit your needs.

`draw()` Canvas items only. Called when node needs redrawing. Use draw* functions here to render custom 2D graphics.

`getconfigurationwarnings()` Return array of strings shown as warnings in editor. Helps catch setup mistakes.

`tostring()` Override how your object appears in print() statements.