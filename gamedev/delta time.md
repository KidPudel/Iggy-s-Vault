delta means "change in"

delta time is a time elapsed since **previous frame** and the current

Because frame is what gets displayed on the screen and between frames we actually run our logic to calculate next frame

this way it accounts for all delays, including rendering, vsync, or external factors like system load.

this allows to take this value and multiple things that move states around with certain speed with delta time, which will make the speed even for every system.