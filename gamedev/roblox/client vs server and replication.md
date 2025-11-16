# The Server-Client Model (The Heart of Roblox)

In Godot, you typically start with a single-player context and add networking later. Roblox is a multiplayer-first platform; every game runs on a server-client architecture, even when you're the only one playing in Studio. The server is the ultimate authority, while clients are essentially windows into the game world.

- **Server**: The owner of the game itself (master data), it manipulates on it directly and securely, it is the *authority*
- **Client**: The receiver/viewer and asker (via shared service and events) for the changes on the server
- **Replicated containers/data**: The exposed data (properties of instances) that are replicatable, the rest of data is locked to specific context (internal to server, internal to client)
- **Replication**: the synchronization of latest master data to the client.

To ensure communication between client and server, we must remote events [[events]]

---

# Mental Model

Think of it like:
The Law: If other players or the server need to rely on the outcome, it *must run on the server*.
The possibly polite suggestion: If it’s just for the player’s own visual experience or for `*`responsiveness and doesn’t matter to others, *it `*`**can** run locally*.
> **NOTE**: Treat all the client checks as the *hints*, not the *truth*.
> A cheater can still send RequestSlideStart with any part or coordinates they like.

In other words:
If we want to replicate changes to all players that we make in our script and be secured, we should use server scripts
- **Server:** verifies, enforces, replicates.
If we want to replicate changes only to one player, meaning it is non sensitive, then we will use local script (client side)
- **Client:** detects, predicts, visualizes, requests.

But with the exception to the replication part - Roblox does _grants_ clients **authority** in very specific, intentional areas:
- Character movement
- Character animation

In other words it server is a truth holder, except for a small set of replicated components where roblox intentionally trusts the client

>TLDR; When to trust the client?
Only for **aesthetic** things: camera movement, UI, cosmetic animation state, or “intent” (e.g. pressing a key).
Never for physical or gameplay-affecting actions.
Even “intent” must be _confirmed_ by the server before consequences happen.


**Client is always lying until proven otherwise.**

---

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

[[replication]]

*   **Server -> Client (Automatic)**: Changes made **by the server** to objects in *replicated containers* (like `Workspace` and `ReplicatedStorage`) are automatically sent to all clients.
    *   *Example*: If the server runs a script that changes a traffic light's color from green to red, every player in the game will see the light turn red.

*   **Client -> Server (Manual)**: This communication is ***almost* never automatic** and must be done explicitly using `RemoteEvents` and `RemoteFunctions`. This is how a client requests an action.
    *   *Example*: A player clicks a button to buy an item. The `LocalScript` detects the click and fires a `RemoteEvent` to the server, saying, "This player wants to buy this item." The server script receives this request, checks if the player has enough money, and if so, deducts the money and grants the item.
- **Client -> Server (Automatic)**: And still some authority is granted for some of the actions to be delegated to the client, because it is unnecessary to load the server with it, whether it is not security dependent or computation heavy like physics. 


# Does it replicate?

| **Component / Behavior**                                                  | **Client can initiate?** | **Will automatic replication happen?** | **Conditions / Notes**                                                                                                           |
| ------------------------------------------------------------------------- | ------------------------ | -------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| Animator → AnimationTracks on _player’s own character_                    | ✅ Yes                    | ✅ Yes                                  | Animator must be properly parented under Humanoid in the character, created or owned by server.  [oai_citation:8‡Developer Forum |
| Character basic movement (walk, jump)                                     | ✅ Yes                    | ✅ Partial                              | Client prediction works; server validates/overrides state.                                                                       |
| Visual effects/particles parented to replicated object                    | ✅ Yes                    | ✅ Possibly                             | The object must be replicated; client must have permission to modify or spawn; replication of children is subject to ownership.  |
| Tool animations loaded via animator for player’s character                | ✅ Yes                    | ✅ Possibly                             | Similar conditions to character animations. Some parts have lag or non-replication issues.  [oai_citation:9‡Developer Forum      |
| Animation on NPC / rig not owned by player                                | ❌ Usually server         | ✅ Yes if server plays                  | If client tries, often fails.  [oai_citation:10‡Developer Forum                                                                  |
| Arbitrary property changes (e.g., leaderstats, game state, inventory)<br> | ❌ No                     | ❌ No                                   | Requires server logic & replication.                                                                                             |

# Example

I want to implement sliding from the slide

## **Client responsibilities:**
- Detect potential slide start (Raycast or Touch event).
- Request slide start from server with relevant info (e.g., hit normal, slide part).
- Play sliding animation when server confirms.
- Optional: perform movement prediction for smoothness (not required at first).

## **Server responsibilities:**
- Validate slide request. Check:
    - The part is truly a slide.
    - The character is actually contacting it (server raycast or GetTouchingParts()).
- Apply physics:
    - Set HipHeight, state changes, friction changes, etc.
    - Apply LinearVelocity (or custom movement loop).
- Track when sliding should end (ground angle changes, speed drops, player jumps, etc.)
- Tell client when to stop animations.

---

## **A cleaner event flow**

Start slide:
1. Client detects potential slide → sends StartSlideRequested(slidePart, surfaceNormal)
2. Server validates → applies movement → fires SlideStarted(character)
3. Client receives that → plays animation

Stop slide:
1. Client or server notices exit condition (speed < threshold, no contact, jump pressed)
2. Server finalizes slide end → fires SlideEnded(character)
3. Client stops animation
  

The server must always be the one who **confirms** start and **finalizes** end. Never rely purely on the client to tell you when physics ends, or exploiters can fly forever.

This is a secure and performant solution