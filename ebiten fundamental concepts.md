- tick: is the time unit for logical update
- frame: is the time unit for rendering/drawing

# game base
[[game loop]] is done using `RunGame` function, that takes our **base of the game**, that implements interface for *handle input, update, and draw*
- `Update` is called every tick, 
- `Draw` is called every frame and takes `screen` image, that is the is the base buffer, that will be displayed on the window (you can think of it as the canvas)
- `Layout` is called when necessary, it takes native device size (actual window size) and returns the game's logical size we have



# image

## "image" philosophy
`ebiten.Image` is the rectangle 2d array set of pixels

In ebiten, everything is image, images created from scratch, from source files, offscreen images (temporary render target), screen itself, because again, it is the set of pixels.
This philosophy is rooted from how the computer graphics are rendered at the low level, where the screen itself is the 2D array of pixels.

> In Ebitengine, **everything** is a **canvas** which you can **put any pixel you want**


```go
imageToDrawInto.DrawImage(anotherImage, &options)
```


# shader
Shader is a program executed in a GPU for executing complex rendering efficiently