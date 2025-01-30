program that translate code to another code as well as other stuff like linking your code [[linker]]

example in c/c++ is clang.

automation tools like `make` that are using Makefile where build targets, dependencies and command to execute are specified, to then build a program with compiler.

And `cmake` is a build system generator that generates a build scripts like Makefile, from file CMakeLists.txt.


---

In odin since there is a package system, and everything builds as one whole entity, we do not need to indicate files separately.

The same as with linking, you just import the package and use it.

But what if we need to use the C projects, for that we need to use [[foreign system]] feature in Odin