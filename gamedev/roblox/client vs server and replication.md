## The Server-Client Model (The Heart of Roblox)

In Godot, you typically start with a single-player context and add networking later. Roblox is a multiplayer-first platform; every game runs on a server-client architecture, even when you're the only one playing in Studio. The server is the ultimate authority, while clients are essentially windows into the game world.

- **Server**: The owner of the game itself (master data), it manipulates on it directly and securely
- **Client**: The receiver/viewer and asker (via shared service and events) for the changes on the server
- **Replicated containers/data**: The exposed data (properties of instances) that are replicatable, the rest of data is locked to specific context (internal to server, internal to client)
- **Replication**: the synchronization of latest master data to the client.

To ensure communication between client and server, we must remote events [[events]]

### The Two Worlds: Server vs. Client

Think of it like:
If other players or the server need to rely on the outcome, it *must run on the server*.
If it’s just for the player’s own visual experience and doesn’t matter to others, *it can run locally*.

In other words:
If we want to replicate changes to all players that we make in our script, we should use server scripts
If we want to replicate changes only to one player, then we will use local script (client side)

#### **The Server**
The server is the **source of truth**. It is the authoritative version of the game world.

*   **Responsibilities**:
    *   **Game State Management**: The server holds the master copy of the game's data, such as player scores, inventories, and the positions of all non-player objects.
    *   **Security & Validation**: It validates all actions requested by clients. A client might *request* to open a door, but the server *decides* if they have the key. This prevents cheating.
    *   **Authoritative Physics**: While clients simulate physics locally for responsiveness, the server has the final say on where objects are.
    *   **Logic Execution**: Code written in `Script` objects runs exclusively on the server.

*   **Context**: Code running on the server has a complete view of the `DataModel` and can access server-only services like `ServerScriptService` and `ServerStorage`.

#### **The Client**
The client is responsible for the initial start and **player's experience**. It presents the game world to the player and captures their input.

*   **Responsibilities**:
    *   **Rendering**: Drawing the 3D world, playing sounds, and displaying the user interface (UI).
    *   **Input Handling**: Capturing keyboard, mouse, touch, and gamepad input from the player.
    *   **Local Simulation**: Running animations and predicting physics for the local player's character to make movement feel responsive and smooth.
    *   **Logic Execution**: Code written in `LocalScript` objects runs on the client.

*   **Context**: A client receives a replicated, filtered copy of the game world from the server. It cannot see or access server-only storage.
> Logic is executed in the "public api front end code" or replicated containers, while startup code for the clients is executed in StarterPlayerScripts ([[Core environments (containers)]])

### FilteringEnabled: The Great Wall
A crucial concept that enforces this separation is **`FilteringEnabled`**. This property of the `Workspace` is now **always on and cannot be disabled**. It is the security foundation of modern Roblox games.

*   **What it does**: `FilteringEnabled` ensures that changes a client makes to the game world are **not automatically replicated** back to the server or to other clients.
*   **Why it's important**: Without it, a cheater could run a `LocalScript` to delete walls, change their health to infinity, or grant themselves items, and those changes would replicate to the server, affecting everyone.
*   **The Rule**: If a client changes a property of an object (e.g., `workspace.MyPart.Color = Color3.new(1, 0, 0)`), only that client will see the color change. The server and all other players will still see the original color. To make a global change, the client must ask the server to do it.

### Replication: How Information Crosses The Wall

Replication is the process by which Roblox automatically synchronizes the game state ***from the server to the clients***.

*   **Server → Client (Automatic)**: Changes made **by the server** to objects in *replicated containers* (like `Workspace` and `ReplicatedStorage`) are automatically sent to all clients.
    *   *Example*: If the server runs a script that changes a traffic light's color from green to red, every player in the game will see the light turn red.

*   **Client → Server (Manual)**: This communication is **never automatic** and must be done explicitly using `RemoteEvents` and `RemoteFunctions`. This is how a client requests an action.
    *   *Example*: A player clicks a button to buy an item. The `LocalScript` detects the click and fires a `RemoteEvent` to the server, saying, "This player wants to buy this item." The server script receives this request, checks if the player has enough money, and if so, deducts the money and grants the item.
