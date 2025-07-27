- middle mouse for rotating around
	- ctrl + middle: zoom in and out
	- shift + middle: move around
	- alt/option+middle mouse: clip the view 
- x: delete
- shift-a: add. just like in godot cmd+a
- shift-d: duplicate an object
- g: grab the object
	- esc/right click: cancel the movement:
	- left click: accept
	- x, y, z: for moving along the chosen axis
	- middle mouse: snap to the nearest axis
- f12: take a photo/render
- n: properties of the selected object. Location, scale, rotation, plugin panels
- t: tool selection
- s: scale
	- supports scaling along axis (could be used in any mode)
- r: rotate
- z: change shading (viewport, material preview, wireframe)
	- shift-z: switch between 2 latest
- ctrl+tab: switch between interaction modes (object, editing, sculpting, etc)
	- tab: to switch between 2 latest
- w: change selection mode: rectangle, lasso
- f9: parameters of the object that are appeared when you create a new object
- right click: commonly used commands on a selected item
- cmd+1: adds a subdivision surface modified. (which splits [[faces]] into smaller parts, making it appear smoother exponentially)
- o: proportional editing: allows to edit along inside the area
	- scroll to increase or decrease the influence area
- alt+left click: edge select, which will select whole edge of connected vertices [[vertex]]
	- with `shift`: selects more
- f2: rename object
- shift+tab: enable snap during transform. can be enabled with different targets
- h: hide
	- alt+h: unhide



Mesh modifiers
- generate subdivision surface modified. (which splits [[faces]] into smaller parts, making it appear smoother exponentially)
- generate solidify: make surface thick

NOTE: When we Add modifier, it is Non-destructive, temporary preview, editable stack, but when we Apply modifier, it Bakes result into real geometry, removes from stack, destructive
NOTE 2: blender applies modifiers from top to bottom, so the order matters. For example, you needed to first subdivision surface of the donut, to then create a copy for icing, and then you apply solidify to make it thick, and on a thicken surface, you apply new subdivision surface for the drips of icing.