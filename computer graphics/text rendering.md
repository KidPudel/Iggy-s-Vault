simplest approach is to map [[texture]] to 2D quads.

# Bitmap fonts
Classic text rendering is bitmap font is a texture that contains selected characters from the font for us to use.
These character symbols are called *glyphs*. Each of them has the coordinate associated with them.
When we need to render specific character, we render specific part of a texture with those coordinates.


# FreeType
Modern text rendering, that is a software library that is able to load fonts -> render them to bitmaps, and provide support for several font-related operations lie macos, java, playstation, linux.
Note: windows is using their own renderer
Appeal is that this library able to load TrueType fonts.
TrueType fonts not defined by pixels, but rather by **mathematical** equations (combinations of splines).

Similar to vector images, the rasterized font images [[rasterization]] can be procedurally generated based on the prefered font height you'd like to obtain them in.
Any size for glyph, without loss of quality.



Font formats store font metrics, which describe the characteristics of each glyph
- width: which is varying to each glyph like . and Q (find approximate min)
- height
- bearing: offset from the origin to the beginning of the character
- advanced width, is the distance to the next glyph
![[Pasted image 20250316113928.png|500]]

bitmap
setting total pixel size of glyph multiplying it by number of glyphs we need getting the square root of it and this will give us 1:1 ration for the bitmap

We need to find a size of a quad 
Clip texture to it
Shift the texture to display the correct glyph