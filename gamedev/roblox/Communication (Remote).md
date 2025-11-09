## Communication: RemoteEvents and RemoteFunctions
[[events]]

These two objects are the core of all client-server interaction in Roblox. They are instances you place in a shared container (almost always `ReplicatedStorage`) to act as secure communication channels.

### `RemoteEvent`: One-Way Communication (Asynchronous)

Think of a `RemoteEvent` as firing a flare or sending a notification. You send a signal with some data, and then your code immediately continues without waiting for a reply. It's a "fire and forget" system.

*   **Asynchronous**: The script that fires the event does not pause or "yield."
*   **Use Cases**:
    *   A client telling the server it has clicked a button, picked up a coin, or finished a task.
    *   The server telling one or more clients to play a sound, show a visual effect, or update their UI.

#### **Communication Flow**

**1. Client → Server**

A `LocalScript` fires the event, and a `Script` on the server listens for it.

*   **LocalScript (Client)**
    ```lua
    local ReplicatedStorage = game:GetService("ReplicatedStorage")
    local MyEvent = ReplicatedStorage:WaitForChild("MyEvent")

    -- Fire the event and send the string "Hello from the client!"
    MyEvent:FireServer("Hello from the client!")
    ```

*   **Script (Server)**
    ```lua
    local ReplicatedStorage = game:GetService("ReplicatedStorage")
    local MyEvent = ReplicatedStorage:WaitForChild("MyEvent")

    -- Listen for the event to be fired from any client
    MyEvent.OnServerEvent:Connect(function(player, message)
        -- NOTE: The 'player' object that fired the event is ALWAYS the first parameter.
        print(player.Name .. " says: " .. message)
    end)
    ```

**2. Server → Client**

A `Script` fires the event to a *specific* client, and a `LocalScript` on that client's machine listens.

*   **Script (Server)**
    ```lua
    local ReplicatedStorage = game:GetService("ReplicatedStorage")
    local MyEvent = ReplicatedStorage:WaitForChild("MyEvent")

    -- Assume 'somePlayer' is a variable holding a specific player object
    -- Fire the event only to that player
    MyEvent:FireClient(somePlayer, "You have been given a reward!")

    -- To fire to ALL clients, you would use:
    -- MyEvent:FireAllClients("A global announcement!")
    ```

*   **LocalScript (Client)**
    ```lua
    local ReplicatedStorage = game:GetService("ReplicatedStorage")
    local MyEvent = ReplicatedStorage:WaitForChild("MyEvent")

    -- Listen for the event to be fired from the server
    MyEvent.OnClientEvent:Connect(function(message)
        -- NOTE: The 'player' object is not passed here because the client already knows who it is.
        print("Server sent a message: " .. message)
    end)
    ```

---

### `RemoteFunction`: Two-Way Communication (Synchronous)

A `RemoteFunction` is like making a phone call and waiting for an answer. You invoke it with some data, and your script will **pause and wait** (yield) until it gets a value returned from the other side.

*   **Synchronous**: The script that calls the function yields until a response is received.
*   **Use Cases**:
    *   A client asking the server if it has enough money to buy an item.
    *   A client requesting its saved data from the server when the game starts.
    *   The server asking a client for information about its screen size or device type.

#### **Communication Flow (Client → Server → Client)**

This is the most common use case. The client requests information, and the server provides it.

*   **LocalScript (Client)**
    ```lua
    local ReplicatedStorage = game:GetService("ReplicatedStorage")
    local MyFunction = ReplicatedStorage:WaitForChild("MyFunction")

    -- Invoke the server and wait for the return value
    -- The code will pause on this line until the server sends a response.
    local serverResponse = MyFunction:InvokeServer("Can I open the door?")

    print("Server responded:", serverResponse) -- Prints "Server responded: Yes you can."
    ```

*   **Script (Server)**
    ```lua
    local ReplicatedStorage = game:GetService("ReplicatedStorage")
    local MyFunction = ReplicatedStorage:WaitForChild("MyFunction")

    -- Set the callback function that runs when a client invokes this RemoteFunction.
    -- This function MUST return a value.
    MyFunction.OnServerInvoke = function(player, request)
        -- NOTE: The 'player' who invoked is always the first parameter.
        print(player.Name .. " is requesting: " .. request)

        -- Some logic to determine the response
        local hasKey = true -- Check player's inventory, etc.

        if hasKey then
            return "Yes you can." -- This value is sent back to the client
        else
            return "No, you need a key."
        end
    end
    ```

### Best Practices & Security

This is the **most important** part of using remotes. Since exploiters can fire any of your remotes with any data they want, you **must never trust the client**.

1.  **Always Validate on the Server**: Before performing any action requested by a client, the server must perform "sanity checks."
    *   Is the player trying to buy an item they can't afford?
    *   Is the player trying to deal more damage than their weapon allows?
    *   Is the player telling the server they moved to a position they couldn't possibly reach?
    *   Check the *type* of data being sent. If you expect a number, make sure it's a number.

2.  **Organize Remotes**: Keep all your `RemoteEvent` and `RemoteFunction` objects in one place, typically a folder inside `ReplicatedStorage`, for easy access and management.

3.  **Use `RemoteFunction`s Sparingly (Client → Server)**: Because an `InvokeServer` call yields the client's script, a poorly written or overloaded server could cause the client to hang or freeze. If the server never returns a value, the client's script will wait forever. Use them only when you absolutely need an immediate answer. Often, you can use two `RemoteEvent`s (one for the request, one for the response) to achieve a similar result asynchronously.

4.  **Use `pcall` with `InvokeClient`**: When calling from the server to the client with a `RemoteFunction`, wrap the call in a `pcall`. This prevents an error if the client disconnects before they can return a value.

Next, we can move on to the **Player & Character API**, which will leverage this communication model.