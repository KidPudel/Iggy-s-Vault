# Service-Based Architecture


*   **Examples of Services:**
    *   **BuildService:** Manages the placement and properties of parts.
    *   **EnemyService:** Controls enemy spawning, behavior, and tracking.
    *   **InventoryService:** Handles player items and data.
    *   **QuestService:** Manages quest logic and progression.

## Simple model: Client, Server, and Utilities
To maintain a clear separation of concerns and enhance security, each service is typically divided into three parts:
*   **Client:** Code that runs on the player's device (e.g., handling user input, displaying UI).
*   **Server:** Code that runs on the Roblox server (e.g., game logic, data validation, security).
*   **Utils (Utilities):** Shared code and data that both the client and server can use (e.g., configuration values, helper functions).

---
# **File and Folder Structure in Roblox Studio**

Here is a practical way to set up your project's hierarchy in the Explorer:

1.  **ReplicatedStorage:**
    *   **Shared:** A folder to hold resources accessible by both the client and server. It is like a public [[API]]
        *   **Services:** Contains the client-side () and utility scripts for replication (communication) for your services specifically. Each service gets its own folder (e.g., `BuildService`). Meaning they manage parts of the system
            *   `BuildServiceClient` (ModuleScript)
            *   `BuildServiceUtils` (ModuleScript)
        * **Classes:** For object-oriented programming structures.
        * **Modules:** For other common shared utility scripts.
        * **Packets:** For the remote [[events]] like API
	* **UI:** Contains the logic for the user interface.
    *   **Packages:** A folder for third-party libraries and packages (e.g., managed with Wally).
    *   **Assets:** A folder for storing models, meshes, sound effects, and other game assets.

2.  **ServerScriptService:**
    *   **Services:** Contains the server-side scripts for your services. This structure mirrors the `Services` folder in `ReplicatedStorage`.
        *   `BuildServiceServer` (ModuleScript)
	- **Starter service script**: initialize and start all the server services
3. **StarterPlayerScript:**
	- **Starter client script**: initialize and start all the client services 

# **Service Code Structure**
*   **Singleton Pattern:** Each service module returns a single table that acts as the instance of that service. There is no `.new()` constructor.
*   **`init()` function:** Most services should have an `init()` function for setup tasks, such as creating necessary objects or running initial code.
*   **Type Checking:** For better code quality and easier debugging, it's recommended to use Luau's strict type checking.

# **Client-Server Communication (Replication)**

For the client and server to communicate, you need to use `RemoteEvents` and `RemoteFunctions`.
*   **Use a Networking Library:** It is highly recommended to use a networking library to handle the creation and management of remote events programmatically. This keeps your project clean and scalable.
    *   **Recommended Open-Source Libraries:** ByteNet, Blink, Zap.

# **The Role of Modules**
Modules provide shared functionality, data, or utilities that other scripts (like services and classes) can depend on. They are distinct from services as they don't manage a whole system but rather provide focused, reusable tools.

- **Organization:** Just like services, modules should be organized into Shared and Server folders. For even better clarity, group them into sub-folders by category.
    - **Example Categories:**
        - **Core:** Essential, game-agnostic utilities.
        - **Game:** Logic specific to your game's mechanics.
        - **Math:** Helper functions for complex calculations.
        - **Platform:** Code for handling different platforms (PC, mobile, console).
- **Purpose:** Keep modules small and focused. They are perfect for things like:
    - Pure data tables (e.g., item configurations, balancing values).
    - Reusable functions (e.g., a custom math library).

# **User Interface (UI) Organization**

The UI is another area that can quickly become disorganized.

*   **Modular Approach:** Break down your UI into modular components.
*   **Top-Level UI Module:** Create a main `UI` ModuleScript in `ReplicatedStorage` that acts as the initializer for all UI elements.
*   **Categorize UI Elements:** Inside the `UI` module, create folders for different types of UI, such as `Menus` and `HUD`. Each feature (e.g., `Inventory`, `LeftSideBar`) should have its own ModuleScript to control its functionality.

---

# The flow

Think of the services in the Shared folder as the **"Public API"** or the **"Front Door"** for your game systems.

Let's use the video's BuildService example to walk through the process:
1. A player clicks their mouse to build something. A LocalScript (client-side logic) in StarterPlayerScripts detects this click.
2. This LocalScript now needs to tell the BuildService what happened. But which part? It can't talk to the server script directly.
3. So, the LocalScript **require()s** the BuildServiceClient module from ReplicatedStorage/Shared/Services/BuildService. This is possible because ReplicatedStorage is visible to the client.
4. The BuildServiceClient is a module that contains client-specific logic. Its main job is to provide a clean way for other client scripts to communicate with the server. When the LocalScript calls BuildServiceClient:attemptBuild(), the BuildServiceClient's job is to fire a RemoteEvent to the server with the necessary information.
5. The **BuildServiceServer** script (hidden away safely in ServerScriptService) is the only one listening for that specific RemoteEvent. It receives the request, validates it (e.g., "Does the player have enough resources?", "Is the player on a build cooldown?"), and then performs the actual, authoritative action of creating the part.
