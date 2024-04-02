This is the foundational pattern in game dev, that is symbolizes running game.

What happens in game processing?
we render a game, we listen for input, updating variable, so to put it in the right order, the game loop is:
1. Handling input and changing variables based on that
2. Update: Applying game logic to altered variables from input
3. Draw: Render the new frame with updated variables
Then repeat and update, by clearing and swapping [[buffer]]s (technique called [[double buffering]])

### architecture
Later when we add more stuff to the game, you start to notice, that we add to much code in the loop
```python
# bad
while True:
	if rl.is_key_pressed(rl.key_space):
		pl_y += ........
	rl.begin_drawing()
	rl.clear_color(rl.white)
	rl.draw_rectangle(pl_x, pl_y, 100, 100, rl.black)
	rl.draw_fps()
	rl.draw_shape(...)
	# ...and so on
	rl.end_drawing()
```

notice, that all objects share the same thing, *handle input*, *update*, *draw*, so this is great idea to separate them

```python
# good
while True:
	player.handle_input()

	player.update()
	enemy.update()

	rl.begin_drawing()
	game.draw()
	player.draw()
	enemy.draw()
	rl.end_drawing()
```