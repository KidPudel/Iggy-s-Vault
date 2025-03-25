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

# Rendering approaches

## Texture atlas approach (Bitmap font)
- Load a font with FreeType library
- Pre-render ([[rasterization]]) all glyphs to a single texture atlas (could become very large if using UTF-8)
- Define a size of a quad with [[UV coordinates]], store them in a buffer
- Map texture with [[UV coordinates]] to the quad
Only efficient for small character sets like ASCII, because it does not involve
Which is a bottleneck if you use full keyword with multiple layouts (languages)


## Software rendering approach (dynamic)
- Load font using FreeType library
- Text is dynamically [[software rendering]](ed)
	- We need to render arbitrary letter "a"
	- Lookup it in font file
	- [[rasterization]] "a" into a grayscale/bitmap using FreeType (CPU)
	- Write bitmap to the [[frame buffer]] (CPU memory)
- send entire [[frame buffer]] to the GPU as asingle image
- no caching
Very slow, but extensible
## Hybrid rendering approach (dynamic)
- Load font using FreeType library
- Text is dynamically [[software rendering]](ed) and cached to GPU glyph atlas/bitmap as typed by [[rasterization]]
	- We need to render arbitrary letter "a"
	- if it isn't in GPU glyph atlas (cached) [[rasterization]] into a small glyph
	- Upload the newly rasterized glyph to the GPU (with growing texture atlas or [[SSBO]] worse approach)
- Store [[UV coordinates]] for the cached glyphs
- Render text using quad's [[UV coordinates]] with right transformations and stuff and cached texture
- If atlas becomes too big, we clear it and start collecting again
Full support of UTF-8
Fast once cached

