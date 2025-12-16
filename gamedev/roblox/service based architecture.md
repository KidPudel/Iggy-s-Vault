# Service-Based Architecture in Roblox

A reference for organizing game logic, managing state, and handling client-server communication.

-----

## Core Concept

just like in [[data-driven design in godot with architecture in mind]], but with additional factor of multiplayer nature

**Service-based architecture** separates game logic into specialized, independent modules called **Services** (server-side) and **Controllers** (client-side).

### The Philosophy

> “Each service owns one domain of game logic. Services communicate through well-defined interfaces. Client and server code are strictly separated. State lives on the server, UI lives on the client.”

### Three Layers

```
┌─────────────────────────────────────────────────────────────────┐
│                        SERVER (Authority)                        │
│                                                                 │
│  ServerScriptService/Services/                                  │
│  ├── InventoryService  (owns player inventories)                │
│  ├── CombatService     (handles damage, death)                  │
│  ├── DataService       (saves/loads player data)                │
│  └── ItemService       (spawns/despawns world items)            │
│                                                                 │
│  - Owns authoritative state                                     │
│  - Validates all actions                                        │
│  - Never trusts client input                                    │
└─────────────────────────────────────────────────────────────────┘
                               │
                               │ RemoteEvents/RemoteFunctions
                               │
┌─────────────────────────────▼─────────────────────────────────┐
│                    CLIENT (Presentation)                       │
│                                                                │
│  StarterPlayer/StarterPlayerScripts/Controllers/               │
│  ├── InventoryController  (UI logic, requests to server)       │
│  ├── InteractionController (detects pickups, calls server)     │
│  └── UIController         (manages GUI state)                  │
│                                                                │
│  - Displays current state                                      │
│  - Sends requests to server                                    │
│  - Updates UI based on server responses                        │
└────────────────────────────────────────────────────────────────┘
                               │
                               │
┌──────────────────────────────▼──────────────────────────────────┐
│                    SHARED (Common Code)                         │
│                                                                 │
│  ReplicatedStorage/Shared/                                      │
│  ├── Definitions/                                               │
│  │   ├── Items.lua        (item templates, like Godot .tres)   │
│  │   └── GameConfig.lua   (constants, settings)                │
│  ├── Classes/                                                   │
│  │   └── ItemInstance.lua (runtime item state)                 │
│  └── Util/                                                      │
│      └── Signal.lua        (custom signal implementation)       │
│                                                                 │
│  - Read-only definitions                                        │
│  - Shared utility code                                          │
│  - No game state (state lives in services)                      │
└─────────────────────────────────────────────────────────────────┘
```

### Why This Order?

1. **Server is authority** — All state changes happen server-side first (of  data that is either built-in and is not automatically handled by the client or it is from our data layer)
2. **Client is dumb** — Only displays what server tells it
3. **Shared is static** — Definitions and utilities, no mutable state
4. **Services are isolated** — Each owns its domain, minimal coupling

-----

## The Three Types of Code

|Type      |Location                             |Purpose                        |Can Access                   |
|----------|-------------------------------------|-------------------------------|-----------------------------|
|**Server**|`ServerScriptService/`               |Game logic, state, validation  |Everything                   |
|**Client**|`StarterPlayer/StarterPlayerScripts/`|UI, input, presentation        |Shared + Client only         |
|**Shared**|`ReplicatedStorage/`                 |Definitions, utilities, classes|Nothing (pure data/functions)|

### Critical Rules

- **Server NEVER requires client code** (impossible anyway)
- **Client NEVER has authoritative state** (always ask server)
- **Shared NEVER requires Server or Client** (must be pure)

-----

## Design Process: Services First

Before writing code, answer these questions:

### 1. What domains exist in your game?

List every system that needs to manage state:

```
Domains:
- Inventory (player items)
- Combat (damage, death, respawn)
- Data (saving/loading)
- Items (world item spawning)
- Quests (tracking progress)
- Economy (shops, trading)
```

Each domain becomes a service.

### 2. What state does each domain own?

```
InventoryService owns:
- inventories: [userId] = {items array}
- capacity per player

CombatService owns:
- active combat sessions
- cooldowns
- damage multipliers

DataService owns:
- player profiles (loaded data)
- session locks
```

### 3. What operations happen on this state?

These become service methods:

```
InventoryService:
- AddItem(player, itemInstance)
- RemoveItem(player, index)
- UseItem(player, index)
- GetInventory(player)
- CanAddItem(player, itemInstance)

CombatService:
- DealDamage(attacker, target, weapon)
- Heal(target, amount)
- ApplyDot(target, damage, duration)
```

### 4. What needs to notify clients?

These become RemoteEvents:

```
InventoryService needs to tell clients:
- InventoryUpdated (full inventory sync)
- ItemAdded (single item)
- ItemRemoved (single item)

CombatService needs to tell clients:
- DamageTaken (for health bar)
- PlayerDied (for death screen)
```

Now you can implement. Structure is clear.

-----

## Implementation

### Directory Structure

```
ServerScriptService/
├── Services/
│   ├── InventoryService.lua
│   ├── CombatService.lua
│   ├── DataService.lua
│   └── ItemService.lua
└── Loader.lua

ServerStorage/
└── Modules/
    └── ProfileTemplate.lua

ReplicatedStorage/
├── Shared/
│   ├── Definitions/
│   │   └── Items.lua
│   ├── Classes/
│   │   └── ItemInstance.lua
│   └── Util/
│       ├── Signal.lua
│       └── TableUtil.lua
└── Remotes/
    └── (RemoteEvents/Functions created by services)

StarterPlayer/StarterPlayerScripts/
├── Controllers/
│   ├── InventoryController.lua
│   ├── InteractionController.lua
│   └── UIController.lua
└── Loader.lua

StarterGui/
└── ScreenGui/
    └── InventoryUI/
```

-----

## Server: Service Pattern

### Basic Service Template

```lua
-- ServerScriptService/Services/InventoryService.lua
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")

local ItemDefinitions = require(ReplicatedStorage.Shared.Definitions.Items)
local ItemInstance = require(ReplicatedStorage.Shared.Classes.ItemInstance)

local InventoryService = {}

-- Private state (only this service can access)
local inventories = {}  -- [userId] = {items array}
local CAPACITY = 20

-- RemoteEvents (created once, reused)
local Remotes = ReplicatedStorage:WaitForChild("Remotes")
local InventoryUpdated = Remotes:WaitForChild("InventoryUpdated")
local RequestPickup = Remotes:WaitForChild("RequestPickup")
local RequestUseItem = Remotes:WaitForChild("RequestUseItem")

--[[
    Initialize player inventory when they join
]]
function InventoryService.InitializePlayer(player)
    inventories[player.UserId] = {}
    
    -- Send initial state to client
    InventoryUpdated:FireClient(player, inventories[player.UserId])
end

--[[
    Add item to player's inventory
    @param player Player - The player
    @param itemInstance ItemInstance - The item to add
    @return boolean - Success
]]
function InventoryService.AddItem(player, itemInstance)
    local inventory = inventories[player.UserId]
    if not inventory then return false end
    
    -- Try stacking first
    if itemInstance.definition.maxStack > 1 then
        for i, existingItem in inventory do
            if InventoryService._CanStack(existingItem, itemInstance) then
                existingItem.quantity += itemInstance.quantity
                InventoryUpdated:FireClient(player, inventory)
                return true
            end
        end
    end
    
    -- Find empty slot
    if #inventory < CAPACITY then
        table.insert(inventory, itemInstance)
        InventoryUpdated:FireClient(player, inventory)
        return true
    end
    
    return false  -- Inventory full
end

--[[
    Remove item from inventory
    @param player Player
    @param index number - Slot index
    @return ItemInstance? - The removed item, or nil
]]
function InventoryService.RemoveItem(player, index)
    local inventory = inventories[player.UserId]
    if not inventory or not inventory[index] then return nil end
    
    local item = table.remove(inventory, index)
    InventoryUpdated:FireClient(player, inventory)
    return item
end

--[[
    Use/consume an item
    @param player Player
    @param index number - Slot index
]]
function InventoryService.UseItem(player, index)
    local inventory = inventories[player.UserId]
    if not inventory or not inventory[index] then return end
    
    local item = inventory[index]
    local itemType = item.definition.itemType
    
    if itemType == "Consumable" then
        -- Apply effect (delegate to other services)
        local PlayerStatsService = require(script.Parent.PlayerStatsService)
        PlayerStatsService.ApplyEffect(player, item.definition.effect)
        
        -- Reduce quantity
        item.quantity -= 1
        if item.quantity <= 0 then
            table.remove(inventory, index)
        end
        
        InventoryUpdated:FireClient(player, inventory)
        
    elseif itemType == "Equipment" then
        -- Equip logic here
    end
end

--[[
    Get player's inventory (for remote function calls)
]]
function InventoryService.GetInventory(player)
    return inventories[player.UserId] or {}
end

-- Private helper
function InventoryService._CanStack(a, b)
    if not a or not b then return false end
    if a.definition.id ~= b.definition.id then return false end
    return a.quantity + b.quantity <= a.definition.maxStack
end

-- Connect remote events
RequestPickup.OnServerEvent:Connect(function(player, itemModel)
    -- Validate item exists and player is close enough
    if not itemModel or not itemModel:IsA("Model") then return end
    if (player.Character.HumanoidRootPart.Position - itemModel.PrimaryPart.Position).Magnitude > 10 then
        return  -- Too far away
    end
    
    -- Get item data from model
    local itemData = itemModel:FindFirstChild("ItemData")
    if not itemData then return end
    
    local itemInstance = ItemInstance.Deserialize(itemData.Value)
    
    if InventoryService.AddItem(player, itemInstance) then
        itemModel:Destroy()  -- Remove from world
    end
end)

RequestUseItem.OnServerEvent:Connect(function(player, index)
    InventoryService.UseItem(player, index)
end)

-- Cleanup on player leave
Players.PlayerRemoving:Connect(function(player)
    inventories[player.UserId] = nil
end)

return InventoryService
```

### Key Points

- **Services are modules** — Return a table of functions
- **State is private** — Only accessible within the service
- **Public API** — Functions that other services can call
- **RemoteEvents** — For client communication
- **Validation** — Always validate client requests (distance checks, ownership, etc.)

-----

## Shared: Definitions and Classes

### Item Definitions (Like Godot Resources)

```lua
-- ReplicatedStorage/Shared/Definitions/Items.lua
return {
    iron_axe = {
        id = "iron_axe",
        name = "Iron Axe",
        description = "A sturdy iron axe",
        icon = "rbxassetid://123456789",
        maxDurability = 100,
        maxStack = 1,
        itemType = "Tool",
        damage = 15,
        attackSpeed = 1.2,
    },
    
    health_potion = {
        id = "health_potion",
        name = "Health Potion",
        description = "Restores 50 HP",
        icon = "rbxassetid://987654321",
        maxStack = 10,
        itemType = "Consumable",
        effect = {
            type = "Heal",
            amount = 50
        }
    },
    
    wheat_seed = {
        id = "wheat_seed",
        name = "Wheat Seeds",
        description = "Plant to grow wheat",
        icon = "rbxassetid://111222333",
        maxStack = 99,
        itemType = "Seed",
        growsInto = "wheat_plant",  -- References plant definition
        growthTime = 120,  -- seconds
    }
}
```

### Item Instance (Runtime State)

```lua
-- ReplicatedStorage/Shared/Classes/ItemInstance.lua
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local ItemDefinitions = require(ReplicatedStorage.Shared.Definitions.Items)

local ItemInstance = {}
ItemInstance.__index = ItemInstance

--[[
    Create new item instance
    @param definitionId string - ID from ItemDefinitions
    @param quantity number - Stack size
    @return ItemInstance
]]
function ItemInstance.new(definitionId, quantity)
    local definition = ItemDefinitions[definitionId]
    if not definition then
        warn("Unknown item: " .. tostring(definitionId))
        return nil
    end
    
    local self = setmetatable({}, ItemInstance)
    self.definition = definition
    self.definitionId = definitionId  -- Store ID for serialization
    self.durability = definition.maxDurability or 0
    self.quantity = quantity or 1
    self.customData = {}  -- For enchantments, custom names, etc.
    
    return self
end

--[[
    Damage the item
    @param amount number
    @return boolean - True if broken
]]
function ItemInstance:Damage(amount)
    if self.durability <= 0 then return true end
    self.durability -= amount
    return self.durability <= 0
end

--[[
    Serialize for saving/network transfer
    @return table
]]
function ItemInstance:Serialize()
    return {
        definitionId = self.definitionId,
        durability = self.durability,
        quantity = self.quantity,
        customData = self.customData
    }
end

--[[
    Deserialize from saved data
    @param data table
    @return ItemInstance
]]
function ItemInstance.Deserialize(data)
    if not data or not data.definitionId then return nil end
    
    local instance = ItemInstance.new(data.definitionId, data.quantity or 1)
    if not instance then return nil end
    
    instance.durability = data.durability or instance.durability
    instance.customData = data.customData or {}
    
    return instance
end

return ItemInstance
```

-----

## Client: Controllers

### Controller Template

```lua
-- StarterPlayer/StarterPlayerScripts/Controllers/InventoryController.lua
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")

local ItemInstance = require(ReplicatedStorage.Shared.Classes.ItemInstance)

local InventoryController = {}

-- Private state (client-side mirror of server state)
local currentInventory = {}

-- UI references (set by UIController)
local inventoryUI = nil

-- RemoteEvents
local Remotes = ReplicatedStorage:WaitForChild("Remotes")
local InventoryUpdated = Remotes:WaitForChild("InventoryUpdated")
local RequestUseItem = Remotes:WaitForChild("RequestUseItem")

--[[
    Initialize controller
]]
function InventoryController.Init()
    -- Listen for inventory updates from server
    InventoryUpdated.OnClientEvent:Connect(function(inventory)
        InventoryController._OnInventoryUpdated(inventory)
    end)
end

--[[
    Request to use an item (sends to server)
    @param index number - Slot index
]]
function InventoryController.UseItem(index)
    RequestUseItem:FireServer(index)
end

--[[
    Get current inventory (client-side cache)
    @return table
]]
function InventoryController.GetInventory()
    return currentInventory
end

--[[
    Private: Handle inventory update from server
]]
function InventoryController._OnInventoryUpdated(inventory)
    -- Deserialize item instances
    currentInventory = {}
    for i, itemData in inventory do
        currentInventory[i] = ItemInstance.Deserialize(itemData)
    end
    
    -- Notify UI to refresh
    if inventoryUI then
        inventoryUI:Refresh(currentInventory)
    end
end

--[[
    Set UI reference (called by UIController)
]]
function InventoryController.SetUI(ui)
    inventoryUI = ui
end

return InventoryController
```

### Interaction Controller (Pickup Items)

```lua
-- StarterPlayer/StarterPlayerScripts/Controllers/InteractionController.lua
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")

local InteractionController = {}

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()

local Remotes = ReplicatedStorage:WaitForChild("Remotes")
local RequestPickup = Remotes:WaitForChild("RequestPickup")

local currentTarget = nil  -- Item player is looking at
local INTERACTION_RANGE = 10

--[[
    Initialize controller
]]
function InteractionController.Init()
    -- Detect items in range
    game:GetService("RunService").Heartbeat:Connect(function()
        InteractionController._UpdateTarget()
    end)
    
    -- Handle input
    UserInputService.InputBegan:Connect(function(input, processed)
        if processed then return end
        if input.KeyCode == Enum.KeyCode.E then
            InteractionController._Interact()
        end
    end)
end

--[[
    Private: Find nearest interactable item
]]
function InteractionController._UpdateTarget()
    local rootPart = character:FindFirstChild("HumanoidRootPart")
    if not rootPart then return end
    
    local nearestItem = nil
    local nearestDistance = INTERACTION_RANGE
    
    -- Find all items in workspace (tagged with CollectionService in real game)
    for _, item in workspace.Items:GetChildren() do
        if item:IsA("Model") and item.PrimaryPart then
            local distance = (rootPart.Position - item.PrimaryPart.Position).Magnitude
            if distance < nearestDistance then
                nearestItem = item
                nearestDistance = distance
            end
        end
    end
    
    -- Update UI prompt
    if nearestItem ~= currentTarget then
        currentTarget = nearestItem
        InteractionController._UpdatePrompt(currentTarget)
    end
end

--[[
    Private: Show/hide interaction prompt
]]
function InteractionController._UpdatePrompt(item)
    local prompt = player.PlayerGui:FindFirstChild("InteractionPrompt")
    if not prompt then return end
    
    if item then
        prompt.Enabled = true
        prompt.Frame.TextLabel.Text = "Press [E] to pick up " .. item.Name
    else
        prompt.Enabled = false
    end
end

--[[
    Private: Attempt to pick up current target
]]
function InteractionController._Interact()
    if currentTarget then
        RequestPickup:FireServer(currentTarget)
    end
end

return InteractionController
```

-----

## Communication: RemoteEvents vs RemoteFunctions

### When to Use Each

|Type              |Use Case                               |Example                                            |
|------------------|---------------------------------------|---------------------------------------------------|
|**RemoteEvent**   |Fire-and-forget, updates, notifications|Inventory updated, damage taken, item spawned      |
|**RemoteFunction**|Request-response, need return value    |Get inventory, check if can afford, validate action|

### RemoteEvent Pattern (Preferred)

```lua
-- SERVER: Send update to client
local InventoryUpdated = Remotes.InventoryUpdated
InventoryUpdated:FireClient(player, inventory)

-- CLIENT: Listen for update
InventoryUpdated.OnClientEvent:Connect(function(inventory)
    -- Update UI
end)

-- CLIENT: Send request to server
local RequestPickup = Remotes.RequestPickup
RequestPickup:FireServer(itemModel)

-- SERVER: Listen for request
RequestPickup.OnServerEvent:Connect(function(player, itemModel)
    -- Validate and process
end)
```

### RemoteFunction Pattern (When Necessary)

```lua
-- SERVER: Setup function
local GetInventory = Instance.new("RemoteFunction")
GetInventory.Name = "GetInventory"
GetInventory.Parent = Remotes
GetInventory.OnServerInvoke = function(player)
    return InventoryService.GetInventory(player)
end

-- CLIENT: Call and wait for response
local inventory = GetInventory:InvokeServer()
print(inventory)
```

**Warning:** RemoteFunctions can timeout and throw errors. Prefer RemoteEvents when possible.

-----

## Save System: DataStoreService

### Profile Template

```lua
-- ServerStorage/Modules/ProfileTemplate.lua
return {
    Coins = 0,
    Level = 1,
    Inventory = {},  -- Array of serialized ItemInstances
    Quests = {},
    Settings = {
        MusicVolume = 0.5,
        SFXVolume = 0.7
    },
    LastSave = os.time()
}
```

### DataService (Using ProfileService)

```lua
-- ServerScriptService/Services/DataService.lua
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local ServerStorage = game:GetService("ServerStorage")
local Players = game:GetService("Players")

local ProfileService = require(ReplicatedStorage.Packages.ProfileService)
local ProfileTemplate = require(ServerStorage.Modules.ProfileTemplate)

local DataService = {}

local ProfileStore = ProfileService.GetProfileStore(
    "PlayerData",
    ProfileTemplate
)

local Profiles = {}  -- [player] = profile

--[[
    Load player data when they join
]]
function DataService.LoadProfile(player)
    local profile = ProfileStore:LoadProfileAsync("Player_" .. player.UserId)
    
    if profile then
        profile:AddUserId(player.UserId)  -- GDPR compliance
        profile:Reconcile()  -- Fill in missing keys from template
        
        profile:ListenToRelease(function()
            Profiles[player] = nil
            player:Kick("Data session released")
        end)
        
        if player:IsDescendantOf(Players) then
            Profiles[player] = profile
            
            -- Initialize other services with loaded data
            local InventoryService = require(script.Parent.InventoryService)
            InventoryService.LoadInventory(player, profile.Data.Inventory)
            
            return true
        else
            profile:Release()
        end
    else
        player:Kick("Failed to load data")
        return false
    end
end

--[[
    Get player's profile
]]
function DataService.GetProfile(player)
    return Profiles[player]
end

--[[
    Save player data (called periodically and on leave)
]]
function DataService.SaveProfile(player)
    local profile = Profiles[player]
    if not profile then return end
    
    -- Collect data from other services
    local InventoryService = require(script.Parent.InventoryService)
    profile.Data.Inventory = InventoryService.SerializeInventory(player)
    
    profile.Data.LastSave = os.time()
    
    -- ProfileService auto-saves, but you can force it
    -- (usually not needed)
end

-- Auto-save every 5 minutes
task.spawn(function()
    while true do
        task.wait(300)  -- 5 minutes
        for player, profile in Profiles do
            DataService.SaveProfile(player)
        end
    end
end)

-- Save on leave
Players.PlayerRemoving:Connect(function(player)
    local profile = Profiles[player]
    if profile then
        DataService.SaveProfile(player)
        profile:Release()
    end
end)

return DataService
```

-----

## Service Initialization: Loader Pattern

### Server Loader

```lua
-- ServerScriptService/Loader.lua
local ServerScriptService = game:GetService("ServerScriptService")
local Players = game:GetService("Players")

local Services = ServerScriptService.Services

-- Load all services
local DataService = require(Services.DataService)
local InventoryService = require(Services.InventoryService)
local CombatService = require(Services.CombatService)
local ItemService = require(Services.ItemService)

-- Initialize services (if they have Init methods)
for _, service in Services:GetChildren() do
    local module = require(service)
    if module.Init then
        module.Init()
    end
end

-- Handle player join
Players.PlayerAdded:Connect(function(player)
    -- Load data first
    if DataService.LoadProfile(player) then
        -- Then initialize other services
        InventoryService.InitializePlayer(player)
        CombatService.InitializePlayer(player)
    end
end)

print("Server services loaded")
```

### Client Loader

```lua
-- StarterPlayer/StarterPlayerScripts/Loader.lua
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")

local player = Players.LocalPlayer
local Controllers = script.Parent.Controllers

-- Load all controllers
local InventoryController = require(Controllers.InventoryController)
local InteractionController = require(Controllers.InteractionController)
local UIController = require(Controllers.UIController)

-- Initialize controllers
InventoryController.Init()
InteractionController.Init()
UIController.Init()

print("Client controllers loaded")
```

-----

## Common Patterns

### Cross-Service Communication

**Option 1: Direct require (services on same side)**

```lua
-- In InventoryService
local CombatService = require(script.Parent.CombatService)

function InventoryService.UseWeapon(player, weapon)
    CombatService.EquipWeapon(player, weapon)
end
```

**Option 2: Signals (loose coupling)**

```lua
-- In InventoryService
local Signal = require(ReplicatedStorage.Shared.Util.Signal)
InventoryService.ItemUsed = Signal.new()

function InventoryService.UseItem(player, index)
    -- ... logic
    InventoryService.ItemUsed:Fire(player, item)
end

-- In QuestService
local InventoryService = require(script.Parent.InventoryService)

InventoryService.ItemUsed:Connect(function(player, item)
    -- Check quest progress
end)
```

### Spawning Items in World

```lua
-- ServerScriptService/Services/ItemService.lua
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local ItemInstance = require(ReplicatedStorage.Shared.Classes.ItemInstance)

local ItemService = {}

local itemTemplate = ReplicatedStorage.Assets.ItemModel  -- Pre-made model

--[[
    Spawn item in world
    @param itemInstance ItemInstance
    @param position Vector3
    @return Model - The spawned item model
]]
function ItemService.SpawnItem(itemInstance, position)
    local itemModel = itemTemplate:Clone()
    itemModel.Name = itemInstance.definition.name
    itemModel.PrimaryPart.Position = position
    
    -- Store item data in model
    local itemData = Instance.new("StringValue")
    itemData.Name = "ItemData"
    itemData.Value = game:GetService("HttpService"):JSONEncode(itemInstance:Serialize())
    itemData.Parent = itemModel
    
    -- Set appearance
    local mesh = itemModel.PrimaryPart:FindFirstChildOfClass("MeshPart")
    if mesh then
        mesh.TextureID = itemInstance.definition.icon
    end
    
    itemModel.Parent = workspace.Items
    return itemModel
end

return ItemService
```

### Client-Side Prediction (Advanced)

For responsive UI, predict server response:

```lua
-- CLIENT: InventoryController
function InventoryController.UseItem(index)
    -- Optimistic update (assume success)
    local item = currentInventory[index]
    if item then
        item.quantity -= 1
        if item.quantity <= 0 then
            table.remove(currentInventory, index)
        end
        inventoryUI:Refresh(currentInventory)  -- Update immediately
    end
    
    -- Send to server
    RequestUseItem:FireServer(index)
    
    -- Server will send authoritative update, which will correct if needed
end
```

-----

## Testing Services

### Unit Testing Pattern

```lua
-- ServerScriptService/Tests/InventoryServiceTest.lua
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local InventoryService = require(game.ServerScriptService.Services.InventoryService)
local ItemInstance = require(ReplicatedStorage.Shared.Classes.ItemInstance)

return function()
    describe("InventoryService", function()
        it("should add item to inventory", function()
            local mockPlayer = {UserId = 12345}
            InventoryService.InitializePlayer(mockPlayer)
            
            local item = ItemInstance.new("health_potion", 1)
            local success = InventoryService.AddItem(mockPlayer, item)
            
            expect(success).to.equal(true)
            
            local inventory = InventoryService.GetInventory(mockPlayer)
            expect(#inventory).to.equal(1)
        end)
        
        it("should stack items", function()
            local mockPlayer = {UserId = 12345}
            InventoryService.InitializePlayer(mockPlayer)
            
            local item1 = ItemInstance.new("health_potion", 5)
            local item2 = ItemInstance.new("health_potion", 3)
            
            InventoryService.AddItem(mockPlayer, item1)
            InventoryService.AddItem(mockPlayer, item2)
            
            local inventory = InventoryService.GetInventory(mockPlayer)
            expect(#inventory).to.equal(1)  -- Should be one stack
            expect(inventory[1].quantity).to.equal(8)
        end)
    end)
end
```

-----

## Security Considerations

### Never Trust the Client

```lua
-- ❌ BAD - Client sends item to add
RequestAddItem.OnServerEvent:Connect(function(player, itemInstance)
    InventoryService.AddItem(player, itemInstance)  -- Client could send hacked item!
end)

-- ✅ GOOD - Client sends item ID, server creates instance
RequestPickup.OnServerEvent:Connect(function(player, itemModel)
    -- Validate item exists in world
    if not itemModel:IsDescendantOf(workspace.Items) then return end
    
    -- Validate distance
    local char = player.Character
    if not char then return end
    local distance = (char.PrimaryPart.Position - itemModel.PrimaryPart.Position).Magnitude
    if distance > 10 then return end  -- Too far
    
    -- Server creates instance from validated data
    local itemData = itemModel:FindFirstChild("ItemData")
    local itemInstance = ItemInstance.Deserialize(
        game:GetService("HttpService"):JSONDecode(itemData.Value)
    )
    
    if InventoryService.AddItem(player, itemInstance) then
        itemModel:Destroy()
    end
end)
```

### Validation Checklist

- [ ] Distance checks (is player close enough?)
- [ ] Ownership checks (does player own this?)
- [ ] Cooldown checks (did they just do this?)
- [ ] Resource checks (can they afford this?)
- [ ] State checks (is this valid right now?)

-----

## Quick Reference

|Want to…           |Do this                                              |
|-------------------|-----------------------------------------------------|
|Create new system  |New service in `Services/` folder                    |
|Add client UI logic|New controller in `Controllers/`                     |
|Share code         |Put in `ReplicatedStorage/Shared/`                   |
|Send data to client|`RemoteEvent:FireClient(player, data)`               |
|Request from server|`RemoteEvent:FireServer(data)` (client)              |
|Save player data   |Use ProfileService in DataService                    |
|Spawn world item   |ItemService.SpawnItem(instance, position)            |
|Load definitions   |`require(ReplicatedStorage.Shared.Definitions.Items)`|

-----

## Checklist

- [ ] Services in `ServerScriptService/Services/`
- [ ] Controllers in `StarterPlayer/StarterPlayerScripts/Controllers/`
- [ ] Definitions in `ReplicatedStorage/Shared/Definitions/`
- [ ] Classes in `ReplicatedStorage/Shared/Classes/`
- [ ] RemoteEvents in `ReplicatedStorage/Remotes/`
- [ ] Server Loader initializes services
- [ ] Client Loader initializes controllers
- [ ] All client requests are validated server-side
- [ ] State lives on server, client is presentation only
- [ ] ProfileService for saving player data

-----

## Common Mistakes

1. **Storing state on client** — Client state is temporary and untrusted. Server is authority.
2. **Not validating remote requests** — Always assume client is hacked. Validate everything.
3. **Circular requires** — Service A requires B, B requires A = infinite loop. Use signals instead.
4. **Forgetting to release profiles** — Always release in `PlayerRemoving` or player is locked out.
5. **Using RemoteFunctions for everything** — They can timeout. Prefer RemoteEvents.
6. **Not handling edge cases** — What if player leaves mid-action? Item is destroyed during pickup?
7. **Mixing client/server code** — Keep them strictly separated or you’ll get confusing errors.

-----

## Next Steps

1. **Study ProfileService** — Industry standard for data saving
2. **Learn Knit framework** — Builds on this pattern with more features
3. **Implement signal library** — For loose coupling between services
4. **Add logging** — Track errors and actions for debugging
5. **Implement anti-cheat** — Detect suspicious actions (too fast, impossible values)

This architecture scales from small games to massive ones. Start simple, add complexity as needed.​​​​​​​​​​​​​​​​