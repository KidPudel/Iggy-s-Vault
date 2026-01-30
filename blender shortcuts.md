# Blender Shortcuts Reference

## Navigation & View

Middle Mouse - Rotate view around cursor

- Ctrl + Middle Mouse - Zoom in/out
- Shift + Middle Mouse - Pan view
- Alt + Middle Mouse - Clip view to axis

Z - Change shading mode (solid/material preview/wireframe)

- Shift + Z - Toggle between last 2 shading modes

` (backtick) - View options like focus on selected object

Alt + Z - Toggle X-ray mode

/ - Toggle isolation mode

N - Properties panel (Location, scale, rotation, plugin panels). Check here if you forgot to apply transformations before baking.

T - Tool selection panel

---

## Selection

A - Select all (all faces/meshes/etc)

- Alt + A - Deselect all

Ctrl + Click - Deselect item

W - Change selection mode (rectangle, circle, lasso)

L - Select linked

Shift + G - Select all similar based on certain property, e.g normal

### Edit Mode Selection

1, 2, 3 - Switch between selection modes (vertices, edges, faces)

Alt + Left Click (edge) - Select edge loop (selects whole edge of connected vertices)

- Shift + Alt + Click - Add another edge loop to selection

Ctrl + Alt + Click (edge) - Select edge ring. An edge ring consists of edges that are not directly connected but are on opposite sides of each other, following a path perpendicular to an edge loop. Fast way to select parallel edges across a mesh surface.

- Shift + Ctrl + Alt + Click - Add another edge ring to selection

---

## Transform Operations

### Movement (G - Grab)

G - Grab/move the object

- Esc / Right Click - Cancel the movement
- Left Click - Accept movement
- X, Y, Z - Move along chosen axis
    - Shift + axis - Exclude that axis (move on other two)
- Middle Mouse - Snap to nearest axis
- Ctrl - Snap to grids (move by 1 meter increments)

G + G - Grab and drag vertex along the edge (edit mode)

### Scaling (S)

S - Scale (supports scaling along axis in any mode)

- Use Shift - Makes transformation slower, hence more accurate
- X, Y, Z - Scale along chosen axis
- S + X + 0 - Flatten/align selected elements (makes all of them 0 at X)
- Alt + S - Reset scale

### Rotation (R)

R - Rotate

- Number keys - Rotate at precise degrees
- X, Y, Z - Rotate around chosen axis
- Alt + R - Reset rotation

### Transform Modifiers

Shift + Tab - Enable snap during transform (can be enabled with different targets)

. (period) - Choose transform pivot point

Shift (during any transform) - Slow/precise mode

---

## Object Mode

Shift + A - Add object (just like Godot Cmd+A)

Shift + D - Duplicate object

Alt + D - Instances object, so that they are referencing the same one (changes applied to all)

X - Delete object

M - Move to collection

Ctrl + J - Join selected objects to active object (yellow one). Note: 2 objects need to be joined to perform bridge edge loops.

Ctrl + P - Assign child object (1st selected) to parent object (2nd selected)

Ctrl + L - Link / Transfer data to other selected objects from target (like modifiers)

H - Hide

- Alt + H - Unhide

Shift + Right Click - Move 3D cursor (allows making it origin in some operations)

---

## Edit Mode

Tab - Switch between Object/Edit mode

- Ctrl + Tab - Full mode menu (object, editing, sculpting, etc)

X - Delete vertex, edge, or face

E - Extrude. Creates new connected geometry from selected elements (vertices, edges, faces) in a direction, usually normal to the selection.

- Alt + E - Special extrude options

I - Inset face. Creates a new, smaller face inside a selected face by insetting its edges, allowing for controlled thickness and depth on a mesh.
- press the I again to inset multiple faces individually

F - Fill edge with face

P - Separate selection into new mesh

M - Merge vertices (this is often a good for removing duplicated vertices)

J - Join vertices to create new edge

Ctrl + B - Bevel selected edges or vertices. Splits the geometry and inserts new edges to soften or round edges (chamfer).

- Ctrl + Shift + B - Bevel vertex specifically
-  ![[Pasted image 20250728120935.png|500]]
- S - while beveling, we can increase amount of loops to make it more round

Ctrl + R - Loop cut. Creates new edge loop with new vertices and new topology, allowing more detailed editing.

K - Knife. Cuts new topology manually.

O - Toggle proportional editing. Allows editing within an area of influence.

- Scroll - Increase or decrease the influence area

U - UV mapping settings

Alt + N - Normals settings (like flipping normals to see inside instead of outside)

---

## Sculpt Mode

F - Change brush size

- Shift + F - Change brush strength

Ctrl + Sculpt - Reverse brush effect

Shift + Sculpt - Smooth out (works with any brush type)

Shift - Invokes smooth brush type in any brush type

Ctrl + I - Invert something like mask brush effect, to paint only on originally masked area

---

## Modifiers & Application

Cmd/Ctrl + 1 - Add subdivision surface modifier (splits faces into smaller parts, making geometry appear smoother exponentially)

Ctrl + A - Apply menu. Use this to bake transformations.

---

## Animation

I - Insert keyframe

T (in Graph Editor) - Choose interpolation mode

Shift + E (in Graph Editor) - Add F-curve extrapolation like cyclic animation

---

## Utility & Quick Access

F2 - Rename object

F3 - Search for action and shortcut

F9 - Parameters of the last created object (appears when you create new object)

F12 - Render/take photo

Right Click - Context menu with commonly used commands on selected item (set origins, etc)

Q - Quick actions

Shift + S - Snap options (like move cursor to selected objects)

Cmd/Ctrl + Alt + S - Incremental save

---

## Key Concepts

Vertex - Single point in 3D space

Edge - Line connecting two vertices

Face - Flat surface bounded by edges

Topology - The arrangement and connectivity of vertices, edges, and faces in a mesh

Edge Loop - Connected edges forming a continuous loop around a mesh

Edge Ring - Parallel edges on opposite sides of each other, perpendicular to edge loops

Extrude - Extends geometry outward from selection, creating new connected elements

Bevel - Chamfers/softens edges by adding new geometry

Inset - Creates smaller face inside selected face