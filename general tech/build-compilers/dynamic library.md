Dynamic or shared library a collection of pre-compiled code that multiple programs can use or rather reference at runtime

Unlike [[static library]], dynamic library is not getting linked at compile time, but rather separate **executable gets loaded/referenced into memory and linked at runtime**, so it is not the part of the executable with [[linker]] 
They typically has `.DLL, .so, .dylib` extension

When you have many applications that use the same library, it is better to have dynamic library, because it is centralized.
If library patches, every applications automatically gets patched version.

Analogy: batter factory, that causes pollution is better than diesel engine where each of them pollutes. If battery factory improve, on a grand schema this improvement automatically affects every car, that they all become cleaner.