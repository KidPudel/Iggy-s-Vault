Built-in function that loads a [[module]], in a cached way (maintains table `package.loaded`), searching it within a predefine file path using patterns and turning it into a real path, and returning the table from that module


```lua
-- mylibrary.lua

local M = {} -- Create a local table to hold the module's functions and data

function M.sayHello(name)
    return "Hello, " .. name .. "!"
end

local function privateHelper()
    -- This function is private to the module and cannot be accessed via require
end

-- Return the table as the module's public interface
return M

```


```lua
-- main.lua

-- Load the 'mylibrary' module. 
-- The argument to require is the name of the file without the .lua extension.
local lib = require("mylibrary")

-- Now you can use the functions provided by the loaded library
local greeting = lib.sayHello("World")

print(greeting)

-- Attempting to call privateHelper() directly would result in an error
-- print(lib.privateHelper()) 

```


```zsh
lua main.lua
```