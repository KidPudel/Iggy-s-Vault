# 1. Finding Objects in the DataModel

[[api for finding objects]]

# 2. Creating and Destroying Instances

[[delete instance]]

The lifecycle of every object in your game.

*   **`Instance.new(className, parent?)`**: Creates a new object. Setting the `.Parent` property is what places it in the game world.
    ```lua
    local newPart = Instance.new("Part")
    newPart.Size = Vector3.new(4, 4, 4)
    newPart.Position = Vector3.new(0, 10, 0)
    newPart.Parent = workspace -- Makes the part appear in the game
    ```
*   **`Instance:Destroy()`**: Removes an instance and all of its descendants, disconnecting all connections. This is the correct way to delete objects.
    ```lua
    task.wait(5)
    newPart:Destroy()
    ```

# 3. Handling Events

Roblox is an event-driven engine. The `:Connect()` method is universal.

*   **`RBXScriptSignal:Connect(function)`**: Binds a function to an event. It returns a `RBXScriptConnection` that can be disconnected later.
    ```lua
    local part = workspace.MyPart
    local connection

    connection = part.Touched:Connect(function(otherPart)
        print(otherPart.Name .. " touched me!")
        connection:Disconnect() -- The event will only fire once
    end)
    ```

# 4. Accessing Core Services

Never access services with `game.Players`. Use `GetService` to prevent issues with name changes and to ensure the service is loaded.
`GetService` is to reliably retrieve the built in [[Services]]

*   **`game:GetService(serviceName)`**: The canonical way to get a reference to a top-level service.
    ```lua
    local Players = game:GetService("Players")
    local ReplicatedStorage = game:GetService("ReplicatedStorage")
    local UserInputService = game:GetService("UserInputService")
    ```

# 5. Client-Server Communication (RemoteEvent)

The lifeblood of a multiplayer game. This demonstrates the most common pattern: client fires, server receives.

*   **`RemoteEvent:FireServer(args...)`** (Client-side)
*   **`RemoteEvent.OnServerEvent:Connect(function(player, args...))`** (Server-side)

    ```lua
    -- In a LocalScript (e.g., inside a GUI button)
    local myEvent = game.ReplicatedStorage:WaitForChild("MyEvent")
    myEvent:FireServer("Damage", 50) -- Send event with arguments

    -- In a Script in ServerScriptService
    local myEvent = game.ReplicatedStorage.MyEvent
    myEvent.OnServerEvent:Connect(function(player, eventType, value)
        -- 'player' is automatically the first argument
        print(player.Name .. " sent event: " .. eventType .. " with value: " .. tostring(value))
    end)
    ```

# 6. Getting the Player and Character

You constantly need to get the player and their in-game avatar.

*   **`Players.LocalPlayer`**: (Client-Only) A direct reference to the client's own `Player` object.
    ```lua
    -- In a LocalScript
    local player = game:GetService("Players").LocalPlayer
    print("My username is " .. player.Name)
    ```
*   **`Player.Character`**: A property on the `Player` object that points to their `Character` model in the `Workspace`.
    ```lua
    -- In a LocalScript, get the local player's character
    local character = player.Character or player.CharacterAdded:Wait()
    ```
*   **`Players:GetPlayerFromCharacter(characterModel)`**: (Server-Side) Gets the `Player` object associated with a given `Character` model.
    ```lua
    -- In a server Script, inside a .Touched event
    local player = game.Players:GetPlayerFromCharacter(otherPart.Parent)
    if player then
        print("The part was touched by a player named " .. player.Name)
    end
    ```

# 7. Moving and Rotating Parts

Directly setting `.Position` will be overridden by physics. Setting `.CFrame` is the correct way to teleport or move anchored objects.

*   **`BasePart.CFrame`**: A property representing both position and orientation.
    ```lua
    local part = workspace.MyPart
    
    -- Teleport the part 20 studs up
    part.CFrame = part.CFrame * CFrame.new(0, 20, 0)
    
    -- Rotate the part 45 degrees on its own Y axis
    part.CFrame = part.CFrame * CFrame.Angles(0, math.rad(45), 0)
    ```

# 8. User Input Handling

The `UserInputService` provides a centralized way to handle all forms of input.

*   **`UserInputService.InputBegan:Connect(function(input, gameProcessed))`**: Fires when any input starts (key press, mouse click, touch).
    ```lua
    local UserInputService = game:GetService("UserInputService")

    UserInputService.InputBegan:Connect(function(inputObject, gameProcessedEvent)
        -- gameProcessedEvent is true if the input was already used by the engine (e.g., clicking on a GUI)
        if gameProcessedEvent then return end

        if inputObject.KeyCode == Enum.KeyCode.E then
            print("Player pressed the E key.")
        elseif inputObject.UserInputType == Enum.UserInputType.MouseButton1 then
            print("Player clicked the left mouse button.")
        end
    end)
    ```

# 9. Re-usable Code (ModuleScripts)

The `require()` global function is used to load `ModuleScript`s, which is essential for organized code.

*   **`require(moduleScriptInstance)`**: Executes a `ModuleScript` and returns the value it provides.
    ```lua
    -- In a ModuleScript named "MyModule"
    local module = {}
    function module.sayHello(name)
        return "Hello, " .. name
    end
    return module

    -- In any other script
    local MyModule = require(game.ReplicatedStorage.MyModule)
    print(MyModule.sayHello("World")) -- Prints "Hello, World"
    ```