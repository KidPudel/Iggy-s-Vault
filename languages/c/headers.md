C header file contains declarations ([[interface]]) and [[macros]] definitions, to be shared between different source files.

You request/access the use of header file by including it in a file with [[preprocessor]] mechanism, via `#include` [[directives in C]]

Headers could be used for:
- System headers, which declared an interface with parts of operating system. You include the header, and forced to supply the definitions for the declarations in order to invoke certain system calls and libraries
- Your own headers, for declaration to have interfaces between sources files. Each time you have a group of related declarations or macros, that are used to define in different parts of a program, it is a good idea to create header file.