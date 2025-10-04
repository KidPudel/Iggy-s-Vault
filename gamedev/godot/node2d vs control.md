`Control` nodes are designed for user interface elements that respond to input, while `Node2D` nodes are for 2D game objects that don't necessarily need UI-specific features. If you're building a UI, use `Control` nodes; if you're creating game elements like sprites, use `Node2D`.

Control nodes make it easy to
- layout elements, especially in a screen space coordinates and Better for resolution independence
- Different support for respond to the input/interaction, which is coming from the UI nature
	- Builtin mouse support
	- etc


Note: container makes the child node say how much it wants size from the parent