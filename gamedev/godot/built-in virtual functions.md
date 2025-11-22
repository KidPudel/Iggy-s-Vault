# Godot Built-in Virtual Functions

**_init()** Constructor called when object is created, before _ready(). Use for initializing variables that don't depend on the scene tree.

**_ready()** Called once when node enters scene tree. Children are guaranteed to exist here. Your main setup function.

**_enter_tree()** Called before _ready() when node enters scene tree. Children may not exist yet. Good for scene-independent initialization.

**_exit_tree()** Called when node leaves scene tree. Cleanup connections and resources here.

**_process(delta)** Called every frame. Delta is time since last frame in seconds. Use for frame-dependent logic like movement.

**_physics_process(delta)** Called every physics frame (default 60 FPS, fixed timestep). Use for physics calculations and rigidbody movement.

**_input(event)** Receives all unhandled input events. Runs before _unhandled_input(). Good for UI and high-priority input.

**_unhandled_input(event)** Receives input events that weren't consumed by UI or _input(). Best for gameplay controls with exception for continuous movement, use _process_

**_unhandled_key_input(event)** Like _unhandled_input() but only for keyboard events. Slightly faster if you only need keys.

**_gui_input(event)**: **scoped to that Control's**, For Control nodes only, UI elements process input here, This is where UI "consumes" events

**_notification(what)** Low-level callback receiving notifications (constants like NOTIFICATION_PAUSED). Use when other callbacks don't fit your needs.

**_draw()** Canvas items only. Called when node needs redrawing. Use draw_* functions here to render custom 2D graphics.

**_get_configuration_warnings()** Return array of strings shown as warnings in editor. Helps catch setup mistakes.

**_to_string()** Override how your object appears in print() statements.