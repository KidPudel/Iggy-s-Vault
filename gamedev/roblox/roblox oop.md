# Code Architecture: Object-Oriented Programming (OOP)

While you can write all your code in simple scripts, a more scalable approach for complex systems is to use Object-Oriented Programming. In Roblox, OOP is implemented using **`ModuleScript`s** and a Lua feature called **metatables**.

This allows you to create "classes" that bundle state (data) and methods (functions) together, which is perfect for things like weapons, pets, custom NPCs, or skill systems.

**Example: A Simple "Coin" Class**

Let's create a `Coin` class that represents a collectable coin in the game. It will handle its own appearance and destruction.

**1. Create a `ModuleScript` (e.g., in `ReplicatedStorage` named "Coin")** singleton

```lua
-- Coin ModuleScript
local Coin = {}
Coin.__index = Coin -- A metatable trick to make methods work on instances

-- Constructor: The function to create a new coin object
function Coin.new(part)
    local newCoin = {}
    setmetatable(newCoin, Coin) -- Make this table behave like a "Coin" object

    newCoin.Instance = part -- The physical Part in the Workspace
    newCoin.Value = 10
    newCoin.Enabled = true

    -- Make the coin hover and spin
    newCoin:Animate()

    return newCoin
end

-- A method of the Coin class
function Coin:Animate()
    -- Logic to make the coin spin would go here (e.g., using a loop in a coroutine)
end

function Coin:Collect()
    if not self.Enabled then return end
    self.Enabled = false

    print("Collected coin worth", self.Value)

    -- Visual feedback (e.g., play a sound, create particles)
    self.Instance:Destroy()
end

return Coin
```

**2. Use the Class from a Server `Script`**

```lua
-- A server script in ServerScriptService
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Coin = require(ReplicatedStorage.Coin)

local allCoins = {} -- A table to keep track of our coin objects

-- Find all parts in the workspace with the "Coin" tag and turn them into Coin objects
for _, part in ipairs(workspace.CoinsFolder:GetChildren()) do
    local coinObject = Coin.new(part)
    table.insert(allCoins, coinObject)

    part.Touched:Connect(function(otherPart)
        local player = game.Players:GetPlayerFromCharacter(otherPart.Parent)
        if player then
            coinObject:Collect()
        end
    end)
end
```

