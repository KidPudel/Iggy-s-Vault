Machine-specific binary or intermediate code produced when a high-level shader language is translated for execution on graphics hardware
This will transform human-readable GLSL into instructions optimized for the target platform
This compiled representation is then embedded in source code (C/Odin) as:
- byte array containing binary shader data and other stuff


This allows to write shader in any shader language, compile it into shader binaries to each platform and choose based on supported renderer backend (macos - metal, windows - directx, linux - vulkan ,etc)


that's done at compile time
the default one is based on your current platform
you can override it if it's supported, e.g. you could switch to opengl from d3d on windows with a flag