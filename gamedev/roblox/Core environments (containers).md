[[Script types and explorer]], [[execution context]], [[Services]], [[DataModel]]

# **Architectural Roles of Core Game Containers**

#### **1. `ServerScriptService` (The Authoritative Secure Backend)**
This service is the secure container for all server-side logic.
*   **Purpose:** To execute all critical, backend game logic that clients must not have access to, such as managing player data, validating purchases, or controlling game state.
*   **Security:** Code and instances stored within `ServerScriptService` are completely invisible and inaccessible to clients. This is the primary defense against exploits, as clients cannot read or modify the server's core scripts.
*   **Authority:** It is the single source of truth for the game. It manages the true state of all data, handles persistence with services like `DataStore`, and validates every request sent from a client before taking action.

#### **2. `ReplicatedStorage` (The Shared Asset & Code Container)**
This service acts as a shared library between the server and all clients, as well as a frontend part of the app (public api and logic).
*   **Replication:** Its primary function is to replicate its contents from the server to every connected client. This makes anything inside it accessible to both server `Script`s and client `LocalScript`s.
*   **Passive Storage:** It is a storage location only. Scripts placed here, typically `ModuleScript`s, **do not run automatically**. They must be explicitly loaded using `require()` by an active script.
*   Role in this Architecture: It holds the "source code" for the client application (UI logic, client-side services, etc.) and any code that needs to be shared between the client and server (utility functions, classes). It also contains the modules that define the network communication contract (e.g., packet definitions).

#### **3. `StarterPlayerScripts` (The Client-Side Loader)**
This is a **template folder**, not a runtime service. Its purpose is to initiate client-side logic for each individual player.
*   **Mechanism:** When a player's character is added to the game, the contents of `StarterPlayerScripts` are **cloned** into a new `PlayerScripts` folder located directly under that player's `Player` object. Every player gets their own independent copy.
*   **Execution:** The engine automatically runs every `LocalScript` it finds inside each player's `PlayerScripts` folder. It does not automatically run `ModuleScript`s.
*   **Role in this Architecture:** Its function is minimized to being an **entry point** or **bootstrapper**. It should contain only one main `LocalScript` whose sole responsibility is to `require` and initialize all the client-side `ModuleScript`s that are stored in `ReplicatedStorage`. This provides a single, controlled start to all client-side code.

---
# Analogy

- **ServerScriptService (The Backend):** This is your Python/Node.js/C# server running on a machine in the cloud. It connects to the database (DataStore) and holds all the secure business logic.
- **ReplicatedContainer (`ReplicatedStorage`, `ReplicatedFirst`, `Workspace`) (The App Code & Public API):** When you visit www.mygame.com, this is the bundle of JavaScript files, components, and assets that your browser downloads. This code knows it can make API calls to endpoints like /api/build or /api/buyItem. The networking package defines these "endpoints."
- **StarterPlayerScripts (The Loader in *Browser* of each client):** Instead, it's the **entry point** or the **bootstrapper** for the frontend. It's like the index.html file of a modern web app. Its only job is to load the actual application. It's the tiny script that kicks everything else off.