C header file contains declarations ([[interface]]) and [[macros]] definitions, to be shared between different source files.

You request/access the use of header file by including it in a file with [[preprocessor]] mechanism, via `#include` [[directives in C]]

A header file (.h) acts as the API of a program or library by exposing declarations — function prototypes, type definitions, macros. We include it in our source code to inform the **compiler** of the symbols we intend to use (function signatures, types, etc.). To actually **use** those functions — i.e., to resolve and link their definitions — we must link against the corresponding **compiled binary** (.so, .a, etc.) using -l, -L, etc. The header and binary are **not inherently bound**; the developer must ensure the binary used in the link step matches the declarations included.

Headers could be used for:
- System headers, which declared an interface with parts of operating system. You include the header, and forced to supply the definitions for the declarations in order to invoke certain system calls and libraries
- Your own headers, for declaration to have interfaces between sources files. Each time you have a group of related declarations or macros, that are used to define in different parts of a program, it is a good idea to create header file.