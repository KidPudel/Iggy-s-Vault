### **The DataModel (Explorer Window)**

In Godot, you have the [[SceneTree]] In Roblox, the equivalent is called the **DataModel** `game`, which you view and interact with through the **Explorer** window in Roblox Studio.

- `game` - to the whole model
- `script` - to the current script

**Core Concepts:**
- **Hierarchical Structure:** The DataModel is a hierarchical tree of objects. Everything that exists in your game—parts, scripts, players, lighting settings, UIs—is an object within this tree.
- **Parent-Child Relationships:** Objects in the tree are organized by parent-child relationships. An object's Parent property determines where it sits in the hierarchy. This is crucial for organization and for finding objects via code.
- **Services:** The top level of the hierarchy consists mainly of "Services." ([[Services]]) These are special singleton objects that manage different aspects of the game. You'll frequently interact with them. Key services include:
    - Workspace: This service contains all the objects that are physically rendered in the 3D world, such as parts, characters, and terrain. It's the bridge between the DataModel and what the player sees.
    - Players: Manages Player objects, which represent each client connected to the game.
    - ServerScriptService: The recommended location for server-side Scripts. Objects here are not visible to clients.
    - ServerStorage: A container for storing objects on the server that you might want to clone and use later. It is not accessible by clients.
    - ReplicatedStorage: A shared container visible to both the server and clients. It's the primary place to store ModuleScripts and RemoteEvents that need to be accessed from both sides.

---

# Instances and Properties

Every single object in the Roblox DataModel inherits from a single base class: Instance. This is the fundamental building block of the engine.

**Key Concepts:**
- **Everything is an Instance:** Whether it's a Part (a physical object), a Script, a Humanoid, or a Folder, it's an Instance. This creates a unified object model.
- **Creating Instances:** You create new instances programmatically using the `Instance.new()` constructor, passing the class name of the object you want to create as a string.
- **Properties:** Instances have properties that define their data and behavior (e.g., a Part has properties like Size, Position, Color, and Material).You can view and edit these in the "Properties" window in Studio or change them via code.
- **Accessing Properties and Children:** You access properties and children using dot notation.

**Example of Creating and Modifying an Instance:**

```lua
-- Create a new 'Part' instance
local myPart = Instance.new("Part")
-- Instance.new("Part", game.Workspace) -- bad practice

-- Set its properties
myPart.Size = Vector3.new(10, 2, 5) -- Vector3 is a data type for 3D coordinates/vectors
myPart.Color = Color3.fromRGB(255, 0, 0) -- Color3 is a data type for RGB colors
myPart.Anchored = true -- An anchored part does not move due to physics

-- Parent the part to the Workspace to make it visible in the game
myPart.Parent = game.Workspace

```

> **NOTE:** parenting the part will add the part to the world and replicate it to the clients. Meaning that the object is now replicatable, and thus all further changes will signal up for replication, therefore we should set all properties before parenting

In this code, game.Workspace is the common way to access the Workspace service directly from a script. The game global variable is a reference to the root of the DataModel.


# Building
The Roblox engine does not guarantee the time or order in which objects are replicated from the server to the client.