# Roblox Containers Reference

## What are Containers?

Containers are special services that **automatically replicate ([[replication]]) their contents** to specific locations when players join or respawn.


[[client vs server and replication]]

---

## Client-Side Containers (LocalScript Territory)

### **StarterGui**

**What it is:** Template storage for player UI. Gets cloned into each player's `PlayerGui` when they join.

- **Replicates to:** `player.PlayerGui`
- **When:** Player joins (once per session)
- **Contents:** ScreenGuis, UI elements, LocalScripts
- **Lifecycle:** Persists until player leaves (unless individual ScreenGui has `ResetOnSpawn = true`)
- **Use for:** Player HUDs, menus, any personal UI

**WaitForChild needed?**

- ❌ No - for siblings/children within same ScreenGui
- ✅ Yes - for player.Character, workspace objects, anything outside container

```lua
-- LocalScript in StarterGui.ScreenGui
local button = script.Parent.Frame.Button -- Safe, same container
local character = game.Players.LocalPlayer.Character or game.Players.LocalPlayer.CharacterAdded:Wait() -- Need to wait
```

---

### **StarterPlayer.StarterPlayerScripts**

**What it is:** Template storage for player scripts. Gets cloned into each player's `PlayerScripts` when they join.

- **Replicates to:** `player.PlayerScripts`
- **When:** Player joins (once per session)
- **Contents:** LocalScripts, ModuleScripts
- **Lifecycle:** Persists entire session
- **Use for:** Player controllers, input handling, client gameplay logic that runs once per session

**WaitForChild needed?**

- ❌ No - for siblings within StarterPlayerScripts
- ✅ Yes - for Character, UI, workspace objects

```lua
-- LocalScript in StarterPlayerScripts
local helperModule = script.Parent.HelperModule -- Safe
local character = game.Players.LocalPlayer.Character or game.Players.LocalPlayer.CharacterAdded:Wait() -- Need to wait
```

---

### **StarterPlayer.StarterCharacterScripts**

**What it is:** Template storage for character scripts. Gets cloned into the character model every time it spawns.

- **Replicates to:** `player.Character`
- **When:** Character spawns/respawns (every death)
- **Contents:** LocalScripts
- **Lifecycle:** Destroyed when character dies, recreated on respawn
- **Use for:** Character-specific logic (animations, abilities, movement modifiers) that resets on death

**WaitForChild needed?**

- ❌ No - for Character parts (Humanoid, HumanoidRootPart, etc.) - cloned together
- ✅ Yes - for UI, workspace objects, anything outside Character

```lua
-- LocalScript in StarterCharacterScripts
local character = script.Parent -- The Character model
local humanoid = character.Humanoid -- Safe, cloned together
local rootPart = character:WaitForChild("HumanoidRootPart") -- Still good practice for critical parts
```

---

## Server-Side Containers (Script Territory)

### **ServerScriptService**

**What it is:** Container for server-only scripts. Scripts here run when the server starts.
This is a secured place

- **Runs on:** Server only (never replicates to client)
- **When:** Server starts
- **Contents:** Scripts, ModuleScripts
- **Lifecycle:** Runs continuously until server shuts down
- **Use for:** Game logic, data management, anything requiring server authority

**WaitForChild needed?**

- ✅ Yes - for workspace objects (they stream in)
- ✅ Yes - for player Characters (spawn at runtime)
- ❌ No - for siblings in ServerScriptService
- ⚠️ Maybe - for ReplicatedStorage/ServerStorage (safer to use it)

```lua
-- Script in ServerScriptService
local helperModule = script.Parent.HelperModule -- Safe, sibling

game.Players.PlayerAdded:Connect(function(player)
    local character = player.Character or player.CharacterAdded:Wait()
    local humanoid = character:WaitForChild("Humanoid") -- Always wait
end)
```

---

### **ServerStorage**

**What it is:** Storage for server-only assets. Never replicates to clients. Not a script container.

- **Accessible:** Server only
- **Use for:** Server-side assets, templates to clone, anything clients shouldn't see

---

## Shared Containers

### **ReplicatedStorage**

**What it is:** Storage for shared assets accessible by both server and client. Not a script container.

- **Accessible:** Server and all clients
- **Replicates:** To clients when they join
- **Use for:** Shared ModuleScripts, RemoteEvents/Functions, shared assets/configs

**WaitForChild needed?**

- ✅ Yes - if script runs early (like from ReplicatedFirst)
- ⚠️ Maybe - in normal scripts (safer to use it, minimal cost)

```lua
-- LocalScript
local replicatedStorage = game:GetService("ReplicatedStorage")
local remoteEvent = replicatedStorage:WaitForChild("GameEvent") -- Safe practice
```

---

### **ReplicatedFirst**

**What it is:** Storage that replicates to clients before anything else. Used for loading screens and critical early setup.

- **Replicates to:** Client first, before all other content
- **When:** Immediately when player connects
- **Contents:** LocalScripts, assets needed for loading screens
- **Use for:** Loading screens, critical initialization that must run before game content loads

**WaitForChild needed?**

- ✅ Yes - for EVERYTHING outside ReplicatedFirst (workspace, StarterGui contents, etc. don't exist yet)

```lua
-- LocalScript in ReplicatedFirst
local loadingGui = script.Parent.LoadingScreen -- Safe, sibling in ReplicatedFirst

-- Everything else needs waiting
local replicatedStorage = game:GetService("ReplicatedStorage")
local config = replicatedStorage:WaitForChild("Config")
```

---

## Quick Decision Tree: Do I Need WaitForChild?

```
Are you accessing something that...

├─ Is a sibling/child in your SAME container?
│  └─ ❌ NO - safe to directly access
│
├─ Is the player's Character or its descendants?
│  └─ ✅ YES - always use WaitForChild
│
├─ Is in Workspace (especially with StreamingEnabled)?
│  └─ ✅ YES - use WaitForChild
│
├─ Was created at runtime by the server?
│  └─ ✅ YES - use WaitForChild
│
├─ Is being accessed from ReplicatedFirst?
│  └─ ✅ YES - everything else loads after, use WaitForChild
│
└─ Is in ReplicatedStorage/ServerStorage?
   └─ ⚠️ SAFER to use WaitForChild (minimal cost)
```

---

## Container Replication Timeline

```
Player Joins
│
├─ [1] ReplicatedFirst → Client
│   └─ LocalScripts run immediately (before anything else exists)
│
├─ [2] StarterGui → player.PlayerGui
│   └─ ScreenGuis + LocalScripts clone in
│
├─ [3] StarterPlayerScripts → player.PlayerScripts  
│   └─ LocalScripts + ModuleScripts clone in
│
├─ [4] Workspace + ReplicatedStorage stream to client
│   └─ May arrive in unpredictable order
│
└─ [5] Character spawns
    └─ StarterCharacterScripts → player.Character
        └─ LocalScripts clone into character
```

**Key insight:** Steps 2-4 happen asynchronously. Your script in step 2 cannot assume step 4 is complete.

---

## Common Mistakes

**❌ Assuming everything exists when your script runs**

```lua
local part = workspace.Map.ImportantPart -- Might not exist yet!
```

**✅ Wait for things outside your control**

```lua
local part = workspace.Map:WaitForChild("ImportantPart")
```

**❌ Using WaitForChild on siblings unnecessarily**

```lua
-- LocalScript in StarterGui.ScreenGui
local frame = script.Parent:WaitForChild("Frame") -- Overkill, Frame is sibling
```

**✅ Direct access for container siblings**

```lua
local frame = script.Parent.Frame -- Safe
```

**❌ Not waiting for Character**

```lua
local character = game.Players.LocalPlayer.Character -- Often nil!
```

**✅ Always wait for Character**

```lua
local character = game.Players.LocalPlayer.Character or game.Players.LocalPlayer.CharacterAdded:Wait()
```

---

## Key Takeaway

**Containers clone their contents atomically as a unit.** You can safely access siblings/children within the same container instance, but always use `WaitForChild` for:

- Anything outside your container
- Runtime-created objects (Characters, server-spawned items)
- Workspace objects (especially with streaming)

When in doubt, use `WaitForChild` - the performance cost is negligible and it prevents race conditions.