label is mark which allows to label a certain piece of code to reference later.
Typically used for labeling for loops and switch statements to break out of outter scoped
```odin
game_loop: for {
	event: sdl.Even
	for sdl.PollEvent(&event) {
		if event.type == sdl.EventType.QUIT {
			break game_loop
		}
	}
}
```