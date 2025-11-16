init file `init.lua` is a essentially an entry point or initialization script similar to the `main` in other languages
In configurations typically used with [[require]] for modular breakdown for organization

Nested `init.lua` (Module Definition): When you use `require()` with a directory name, Lua is programmed to specifically look for an `init.lua` file inside that directory and execute it. This makes the nested `init.lua` the entry point _for that specific module only_.
