Static or *statically linked* library is a collection of pre-compiled external code that a our program will embed in itself.

After compiler generates [[object files]] compiler invokes [[linker]] and this library is getting linked at a compile time, meaning that their code becomes a part of the final executable, unlike [[dynamic library]]

They typically has `.lib .a` extension



Build static library
```zsh
clang -c sqlite3/sqlite3.c -o sqlite3/sqlite3.o
```
but This is **not yet executable or linkable** on its own—just raw compiled code.

archive tool to create a static library 
```zsh
ar rcs sqlite3/libsqlite3.a sqlite3/sqlite3.o
```


or use make if you have make file, which is cli tool to make like easier
