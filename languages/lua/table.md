the only data structure for storing collections

Key characteristics of Lua tables
- **Associative arrays:** Tables are associative, meaning they can be indexed with any value, not just numbers. This includes numbers, strings, and even other tables (with the exception of `nil` and `NaN`).
- **Dynamic size:** Tables have no fixed size. They can grow and shrink as needed at runtime.
- **Created by constructor `{}` :** You create a table using a constructor, which is a pair of curly braces. 

---
# How tables work in Lua
# Creating tables
To create a table, you use a constructor:

```lua
-- Create an empty table
local myTable = {}
```

# Table as an array

When you store values in a table without specifying a key, Lua automatically assigns numerical keys starting from 1. 

```lua
-- Table as an array (a sequence of values)
local fruits = {"apple", "banana", "cherry"}

-- Accessing elements using numerical indices (starts at 1)
print(fruits[1])  -- Output: apple
print(fruits[3])  -- Output: cherry
```

# Table as a dictionary

You can also use string-based keys to create a dictionary-like structure. 
```lua
-- Table as a dictionary (key-value pairs)
local fruitColors = {
  apple = "red",
  banana = "yellow"
}

-- Accessing elements using string keys
print(fruitColors["apple"])  -- Output: red
-- You can also use dot notation for valid string keys
print(fruitColors.banana)   -- Output: yellow
```

# Table as an object

Tables can be used to create objects by combining properties (data) and methods (functions) within a single structure. 

```lua
local player = {
  health = 100,
  name = "Hero",

  -- A method defined within the table
  takeDamage = function(self, amount)
    self.health = self.health - amount
    print(self.name .. " has " .. self.health .. " health remaining.")
  end
}

-- Call the method
player.takeDamage(player, 20)  -- Output: Hero has 80 health remaining.
```

# Important note on reference

Tables are objects and are passed by reference. This means that when you assign a table variable to another variable, you are creating another reference to the **same table**, not a copy. 

```lua
local a = {x = 10}
local b = a -- `b` references the same table as `a`
b.x = 20  -- Changing `b` also changes `a`
print(a.x) -- Output: 20
```
# . vs :
```lua
local myTable = {}
function myTable.someFunc(arg1)
  print(arg1)
end

myTable.someFunc("Hello") -- Output: Hello
```
just for organaization

```lua
local myTable = {}
function myTable:someMethod(arg1)
  print(self) -- The table itself is now the 'self' parameter
  print(arg1)
end

myTable:someMethod("World") -- Equivalent to myTable.someMethod(myTable, "World")
-- Output:
-- table: 0x...
-- World
```

for automatically passing self (table itself), meaning it is for [[oop]] like code.