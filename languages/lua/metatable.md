Metatable are just [[table]] of functions called [[metamethod]] attached to the data inside of a [[table]]
These metamethods are fired when a certain action is used on a table

Metatables are used to change the behavior of a table.

When you normally index a table to grab one of it's values:
`print(tbl.hello)`
The table will just look inside of itself to find it, but if the table has a metatable with the __index metamethod, it will call
`__index(table, key)`
instead of indexing the table. This function could return any kind of value you have programmed it to return. This can allow many advanced behaviors such as object oriented programming.


## Manipulate metatables

The two primary functions for adding and finding a table's metatable are [setmetatable()](https://create.roblox.com/docs/reference/engine/globals/LuaGlobals#setmetatable) and [getmetatable()](https://create.roblox.com/docs/reference/engine/globals/LuaGlobals#getmetatable).

```lua
local x = {}
local metaTable = {} -- metaTables are tables, too!
setmetatable(x, metaTable) -- Give x a metatable called metaTable!
print(getmetatable(x)) --> table: [hexadecimal memory address]
```

# Usage

```lua
local metatable = {
	__unm = function(t) -- __unm is for the unary - operator
    local negated = {}
  	for key, value in t do
  		negated[key] = -value -- negate all of the values in this table
  	end
  	return negated -- return the table
	end
}

local table1 = setmetatable({10, 11, 12}, metatable)
print(table.concat(-table1, "; ")) --> -10; -11; -12
```

```lua
local metatable = {
	__index = {x = 1}
}

local t = setmetatable({}, metatable)
print(t.x) --> 1
```
__index was fired when x was indexed in the table and not found. Luau then searched through the __index table for an index called x, and, finding one, returned that.

Now you can easily do that with a simple function, but there's a lot more where that came from. Take this for example:


```lua
local metatable = {
	__call = function(t, param)
	local sum = {}
		for i, value in ipairs(t) do
			sum[i] = value + param -- Add the argument (5) to the value, then place it in the new table (t).
		end
		return unpack(sum) -- Return the individual table values
	end
}

local t = setmetatable({10, 20, 30}, metatable)
print(t(5)) --> 15 25 35
```