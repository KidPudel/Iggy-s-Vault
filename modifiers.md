Mesh modifiers
- generate subdivision surface modified. (which splits [[faces]] into smaller parts, making it appear smoother exponentially)
- generate solidify: make surface thick
- deform shrinkwrap: wraps one object into target object
- deform simple: deforms a shape (or selected group) by twisting, bending, rotating etc

NOTE: When we Add modifier, it is Non-destructive, temporary preview, editable stack, but when we Apply modifier, it Bakes result into real geometry, removes from stack, destructive
NOTE 2: blender applies modifiers from top to bottom, so the order matters. For example, you needed to first subdivision surface of the donut, to then create a copy for icing, and then you apply shrinkwrap to wrap around and then solidify to make it thick, and on a thicken surface, you apply new subdivision surface for the drips of icing.