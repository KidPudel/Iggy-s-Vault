An **execution context** is the specific environment where your code is processed and run. In Roblox, the entire architecture is built on two primary contexts that are separate and isolated from each other: the **Server** and the **Client**.

---
#### **1. The Server Context**
*   **Where it Runs:** On Roblox's official cloud servers.
*   **Its Role:** To be the single **Source of Truth**. The server is the authoritative manager of the game. Its responsibilities include:
    *   Managing the game's official state (player data, NPC positions, game rules).
    *   Ensuring security and validating all requests from clients.
    *   Running the authoritative physics simulation.
*   **Scripts Used:** Code in **`Script`** objects runs in the server context. These are typically stored in the secure **`ServerScriptService`**.
*   **View of the Game:** The server has a complete and unfiltered view of the entire game's `DataModel`.
*   **Trust Level:** **Trusted**. The server is the authority and its own logic is considered secure.

---
#### **2. The Client Context**
*   **Where it Runs:** On each player's individual device (PC, phone, console). Every player has their own separate client context.
*   **Its Role:** To manage **the player's experience**. The client's job is to present the game to the player and capture their input. Its responsibilities include:
    *   Rendering the 3D world, UI, and visual effects.
    *   Capturing keyboard, mouse, and touch input.
    *   Playing sounds.
*   **Scripts Used:** Code in **`LocalScript`** objects runs in the client context. These are typically stored in places that replicate to the client, like `StarterPlayerScripts` or `StarterGui`.
*   **View of the Game:** The client has a **replicated, filtered copy** of the game world. It cannot see or access server-only containers like `ServerScriptService`.
*   **Trust Level:** **Untrusted**. Because code on the client can be manipulated by exploiters, the server must never trust any information or request that comes from a client without validating it first.

---
#### **The `ModuleScript` Context Rule (Crucial)**

A `ModuleScript` is a library of code that does not run on its own. Its context is determined by who asks for it.

> **A `ModuleScript` is executed in the context of the script that `require()`s it.**

*   If a **`LocalScript`** (Client) calls `require()`, the module's code runs on the **Client**.
*   If a **`Script`** (Server) calls `require()`, the module's code runs on the **Server**.

This is why you cannot "execute server logic" from a `ModuleScript` in `ReplicatedStorage` just by calling it from a `LocalScript`. The act of calling it from the client *forces* the code to run on the client. To get the server to run its logic, you must use a `RemoteEvent`.

---
### **Summary Table**

| Aspect                 | Server Context                                | Client Context                                |
| :--------------------- | :-------------------------------------------- | :-------------------------------------------- |
| **Runs On**            | Roblox Cloud Servers                          | Player's Device                               |
| **Role**               | Source of Truth, Authority                    | Player Experience, Presentation               |
| **Script Type**        | `Script`                                      | `LocalScript`                                 |
| **Game View**          | Complete, Unfiltered                          | Replicated, Filtered                          |
| **Trust Level**        | Trusted                                       | **Untrusted**                                 |
| **`ModuleScript` Use** | `require()` runs the module on the **Server** | `require()` runs the module on the **Client** |
