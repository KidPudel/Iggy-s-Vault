`DataStores` are Roblox's built-in key-value database system. They allow you to save and retrieve data associated with a key, which is almost always a player's unique `UserId`. This is how you save a player's currency, inventory, level, or any other game progress.

**Key Characteristics**:
*   **Scope**: Data is specific to each game. You cannot access a `DataStore` from another game unless you use the advanced `GlobalDataStore`.
*   **Data Format**: You can save Luau data types such as strings, numbers, booleans, and tables. You cannot save instances directly.
*   **Security**: `DataStore` operations can **only be performed by server `Script`s**. This is a critical security measure to prevent clients from modifying their own save data.

### The `DataStoreService`

The first step is to get the service that manages all `DataStores`.

```lua
local DataStoreService = game:GetService("DataStoreService")
```

From this service, you can then get a specific named `DataStore`. It's good practice to name your `DataStore` to avoid mixing different types of data.

```lua
-- Get a DataStore named "PlayerData_V1". It's wise to version your data stores.
local playerDataStore = DataStoreService:GetDataStore("PlayerData_V1")
```

### Core Functions: Get, Set, and Update

There are three primary functions you will use to interact with a `DataStore`. Because these are network calls to an external database, they can sometimes fail. Therefore, you **must always wrap them in a `pcall()`** (Protected Call). A `pcall` safely executes a function and catches any errors that occur without crashing the script.

`pcall` returns two values:
1.  `success`: A boolean that is `true` if the function ran without errors, and `false` if it failed.
2.  `result`: If successful, this is the value returned by the function. If it failed, this is the error message.

#### 1. `GetAsync(key)`
This function retrieves the data associated with the given key.

**Example: Loading a Player's Data**
```lua
local Players = game:GetService("Players")

Players.PlayerAdded:Connect(function(player)
    local playerKey = "Player_" .. player.UserId -- e.g., "Player_1234567"

    local success, data = pcall(function()
        return playerDataStore:GetAsync(playerKey)
    end)

    if success then
        if data then
            -- Data exists for this player, load it
            print("Loaded data for " .. player.Name, data)
            -- e.g., set the player's cash value from data.Cash
        else
            -- No data exists, this is likely a new player
            print(player.Name .. " is a new player.")
            -- e.g., give them default starting cash
        end
    else
        -- An error occurred while trying to get the data
        warn("Could not get data for " .. player.Name .. ". Error: " .. data)
    end
end)
```

#### 2. `SetAsync(key, data)`
This function saves data to the store, completely **overwriting** any existing data at that key.

**Example: Saving a Player's Data**
```lua
local function saveData(player)
    local playerKey = "Player_" .. player.UserId

    local dataToSave = {
        Cash = 100, -- Get the player's actual cash value
        Level = 5   -- Get the player's actual level
    }

    local success, errorMessage = pcall(function()
        playerDataStore:SetAsync(playerKey, dataToSave)
    end)

    if success then
        print("Successfully saved data for " .. player.Name)
    else
        warn("Failed to save data for " .. player.Name .. ". Error: " .. errorMessage)
    end
end

-- Hook this function to the PlayerRemoving event
Players.PlayerRemoving:Connect(saveData)
```

#### 3. `UpdateAsync(key, function(oldData) ... end)`
This is the safest and most powerful way to save data. It first *gets* the current data, then runs a function to transform it, and finally saves the new result. This prevents **race conditions**, where two different game servers might try to save data for the same player at once.

`UpdateAsync` is better than `SetAsync` because its operation is atomic. It effectively "locks" the player's data, applies the change, and then unlocks it.

**Example: Updating a Player's Cash**
```lua
function addCash(player, amount)
    local playerKey = "Player_" .. player.UserId

    local success, updatedData = pcall(function()
        return playerDataStore:UpdateAsync(playerKey, function(oldData)
            -- 'oldData' is the current value associated with the key (or nil if none exists)
            local newData = oldData or { Cash = 0, Level = 1 } -- Create new data if they're a new player

            newData.Cash = newData.Cash + amount

            return newData -- The value returned here is what will be saved
        end)
    end)

    if success then
        print("Successfully updated cash for " .. player.Name .. ". New balance:", updatedData.Cash)
    else
        warn("Failed to update cash for " .. player.Name)
    end
end
```

### Best Practices

1.  **Use `UserId`, Not `Name`**: Player names can change, but their `UserId` is permanent. Always use the `UserId` to create your `DataStore` keys.
2.  **Save in Multiple Places**: Don't rely only on `PlayerRemoving`. A server might crash, in which case `PlayerRemoving` might not fire.
    *   **Auto-Save**: Implement a loop that saves every player's data every few minutes.
    *   **Bind to Close**: Use `game:BindToClose()` to run a final save for all players when the server is shutting down.
3.  **Manage `Studio` Access**: In Studio, API access to `DataStores` is disabled by default. You must go to **Game Settings > Security** and enable **"Enable Studio Access to API Services"** to test `DataStore` functionality.
4.  **Avoid Throttling**: Roblox places limits on how often you can make `DataStore` requests. Don't save data every time a player's cash changes. Instead, save it periodically or when they leave.