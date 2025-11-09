Events in Roblox are signals that fire when a specific action occurs. You use the universal **`:Connect()`** method to run a function whenever an event fires.

[[Communication (Remote)]]

---

### 1. Instance Events (Object-Specific)
These events are part of specific objects and fire in response to changes in their state or interactions.

*   **`Instance.Touched`**
    *   **What:** Fires when one `BasePart` collides with another.
    *   **When to Use:** Detecting when a player walks on a part, a bullet hits a wall, or for any physics-based trigger.

*   **`Instance.ChildAdded` / `Instance.ChildRemoved`**
    *   **What:** Fires when a new child is parented to (or removed from) an instance.
    *   **When to Use:** Detecting when a tool is equipped to a character or an item is added to a folder.

*   **`Instance:GetPropertyChangedSignal("PropertyName")`**
    *   **What:** A precise event that fires only when a *specific property* of an instance changes.
    *   **When to Use:** Running code exactly when a part's `.Color` changes, a `Humanoid`'s `.Health` changes, or an `IntValue`'s `.Value` changes.

---

### 2. Input Events (Client-Only)
These are fired by `UserInputService` and are the primary way to handle player input. They only work in **`LocalScript`s**.

*   **`UserInputService.InputBegan`**
    *   **What:** Fires when any input starts (a key is pressed, a mouse button is clicked, a finger touches a screen).
    *   **When to Use:** This is your main tool for all player actions like jumping, shooting, opening menus, or interacting with the world.

---

### 3. Game Loop Events
Fired by `RunService`, these events allow you to run code on every single frame of the game.

*   **`RunService.Heartbeat` (Server-Side)**
    *   **What:** Fires every frame on the **server**, immediately after the physics simulation step.
    *   **When to Use:** Ideal for any continuous server-side game logic, like moving platforms, updating NPC behavior, or running a game timer.

*   **`RunService.RenderStepped` (Client-Side)**
    *   **What:** Fires every frame on the **client**, just before the scene is rendered.
    *   **When to Use:** Essential for tasks that must be perfectly in sync with the player's screen, such as custom camera manipulation or character updates that need to feel instantly responsive.

---

### 4. Communication Events (Client-Server)
These are `Instance`s you create (usually in `ReplicatedStorage`) to send signals between the server and clients.

*   **`RemoteEvent` (One-Way / "Fire and Forget")**
    *   **What:** Sends a one-way, asynchronous signal. The script that fires it continues running immediately without waiting.
    *   **When to Use:**
        *   **Client → Server:** A client telling the server it performed an action (e.g., "I clicked the 'Buy Item' button").
        *   **Server → Client(s):** The server telling one or more clients to do something visual (e.g., "Play an explosion effect at this position").

*   **`RemoteFunction` (Two-Way / "Request and Wait for Response")**
    *   **What:** Sends a two-way, synchronous request. The script that calls it **pauses and waits** until it receives a value back from the other context.
    *   **When to Use:** A client asking the server for information it needs *right now*. For example, "Can I afford this item?" The client's script will wait for the server to return `true` or `false`. Use sparingly from client to server to avoid freezing if the server is slow to respond.

---

### 5. Custom Script Events
Used for communication between scripts **within the same context**.

*   **`BindableEvent`**
    *   **What:** A signal that allows scripts to communicate *without* crossing the client-server boundary.
    *   **When to Use:**
        *   **Server → Server:** Allowing two different server-side services to communicate with each other.
        *   **Client → Client:** Allowing two different `LocalScript`s to communicate.