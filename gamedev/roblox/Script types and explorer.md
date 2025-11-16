# Script Types & Their Execution Contexts

The three fundamental script types you will use are `Script`, `LocalScript`, and `ModuleScript`. Each has a distinct and non-interchangeable role dictated by the server-client architecture.

### `Script` (Server-Side Logic)

This is your workhorse for all authoritative game logic. Code inside a `Script` object runs **only on the server**. Clients cannot see or access the source code of these scripts.

*   **Execution Context**: Server
*   **Primary Purpose**:
    *   Managing the game loop and rules.
    *   Awarding points, managing currency, and handling player data.
    *   Controlling non-player characters (NPCs).
    *   Validating requests from clients and making secure changes to the game state.
*   **Common Locations**:
    *   **`ServerScriptService`**: The ideal and most secure location. Its contents are **never** replicated to clients. This is where 99% of your server logic should live.
    *   **`Workspace`**: Placing a script here (e.g., inside a `Part` or a `Model`) is also common, especially when the script's logic is directly tied to that object. For example, a script inside a treasure chest part that handles giving a reward when touched.

**Example Use Case**: A script in `ServerScriptService` that gives a player 10 points every 60 seconds.

```lua
-- Located in ServerScriptService
local Players = game:GetService("Players")

while wait(60) do
    for _, player in ipairs(Players:GetPlayers()) do
        -- Assuming the player has a 'leaderstats' folder with a 'Points' value
        local leaderstats = player:FindFirstChild("leaderstats")
        if leaderstats then
            local points = leaderstats:FindFirstChild("Points")
            if points then
                points.Value = points.Value + 10
            end
        end
    end
end
```

### `LocalScript` (Client-Side Logic)

A `LocalScript` runs **on the player's client**. Every player runs their own independent copy of the game's `LocalScript`s. This is where you create an interactive and responsive experience for the player.

*   **Execution Context**: Client
*   **Primary Purpose**:
    *   **Input Handling**: Detecting mouse clicks, keyboard presses, and UI interactions.
    *   **UI Management**: Manipulating `ScreenGui` elements, updating text labels, and playing animations.
    *   **Camera Control**: Creating custom camera systems.
    *   **Visual/Audio Effects**: Creating effects like particles, sounds, or screen shakes that only the local player needs to experience immediately.
*   **Where it will run**: A `LocalScript` will *only* execute if it is a descendant of one of the following objects:
    *   **`StarterPlayer`**: Scripts placed in `StarterPlayerScripts` will run once for each player when they join the game.
    *   **`StarterCharacterScripts`**: Scripts here are copied into each player's `Character` model when it spawns. Ideal for character-specific logic like custom movement.
    *   **`StarterGui`**: Scripts placed here are copied into each player's `PlayerGui` and are perfect for managing the UI.
    *   **ReplicatedFirst**: Code here runs before anything else is replicated, used for creating custom loading screens.
    *   A player's `Backpack` or `PlayerGui`.

**IMPORTANT NOTE:** Since the client scripts in client [[roblox containers with replication]] will all run only once when the player enters the game, meaning `game.Loaded` event ([[events]]) is fired which signals that the game finishes loading, then there is no point to `WaitForChild()` [[Roblox APIs]]

**Example Use Case**: A `LocalScript` inside a `TextButton` that detects a click and fires a `RemoteEvent` to the server.

```lua
-- Located inside a TextButton, which is inside a ScreenGui
local button = script.Parent
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local buyItemEvent = ReplicatedStorage:WaitForChild("BuyItemEvent")

button.MouseButton1Click:Connect(function()
    -- Tell the server that this client wants to buy the "Sword"
    buyItemEvent:FireServer("Sword")
end)
```

### `ModuleScript` (Reusable Code Libraries)

A `ModuleScript` is a module, which means **It does not run on its own.** Instead, it acts as a library of functions and data that can be loaded and used by other scripts. It's the Roblox equivalent of a class, a singleton, or a module in other languages.

*   **Execution Context**: *Inherited*. A `ModuleScript` runs in the context of the script that `require()`s it.
    *   If a server `Script` requires it, the module code runs on the **server**.
    *   If a `LocalScript` requires it, the module code runs on the **client**.
*   **Key Feature**: A `ModuleScript` must return **exactly one value**. This is typically a table containing a set of functions or a metatable for Object-Oriented Programming (OOP). The engine caches this return value; the first time a script in a given context requires a module, its code runs, and the result is saved. Subsequent `require()` calls from that same context will receive the already-computed result.
*   **Primary Purpose**:
    *   **Code Reusability / Don't Repeat Yourself (DRY)**: Centralize functions used by multiple scripts.
    *   **Object-Oriented Programming (OOP)**: They are the standard way to implement classes and objects.
    *   **Configuration Management**: Store shared settings or configuration data.
*   **Common Locations**:
    *   **`ReplicatedStorage`**: Ideal for modules that need to be accessed by **both** the server and the client (e.g., a module with game settings or a custom character class).
    *   **`ServerScriptService`**: For modules that are only used by the server (e.g., a data-saving module).

**Example `ModuleScript` (e.g., in ReplicatedStorage named "GameUtils")**:

```lua
-- GameUtils ModuleScript
local GameUtils = {}

GameUtils.GOLD_PER_WIN = 100

function GameUtils.GetPlayerFromCharacter(character)
    local Players = game:GetService("Players")
    return Players:GetPlayerFromCharacter(character)
end

return GameUtils
```

**Using the `ModuleScript` from a server `Script`**:

```lua
-- A server script
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local GameUtils = require(ReplicatedStorage.GameUtils)

print("Gold per win is:", GameUtils.GOLD_PER_WIN) -- Prints 100

workspace.SomePart.Touched:Connect(function(hit)
    local character = hit.Parent
    local player = GameUtils.GetPlayerFromCharacter(character)
    if player then
        print(player.Name .. " touched the part!")
    end
end)
```
