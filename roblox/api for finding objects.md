You'll spend most of your time getting references to objects (as usual). These two methods are fundamental.

**`Instance:WaitForChild(childName, timeout?)`**: Pauses the script's execution until the specified child exists. **This is crucial for client-side scripts** *waiting for the server to replicate objects* ([[replication]]).
```lua
-- In a LocalScript, guaranteed to work even if "MyPart" hasn't loaded yet
local part = workspace:WaitForChild("MyPart")
```

**IMPORTANT NOTE:** Since the client scripts in client [[roblox containers with replication]] will all run only once when the player enters the game, meaning `game.Loaded` event ([[events]]) is fired which signals that the game finishes loading, then there is no point to `WaitForChild()` [[Roblox APIs]]

When to use it?
- For everything that is created during runtime from the server, because [[replication]]
- For every model in a Workspace which has `StreamingEnabled` turned on, because if part that is far away from the player's character may not be streamed to the client, potentially causing scripts to break when indexing objects that do not yet exist on the client.
- Run from the Local script that is inside `ReplicatedFirst` and referencing for something that is outside of it, because we are running from the first things that replicate.
- **Character loading** - The character and its descendants are runtime-created.
When not to?
- Read about LocalScripts [[Script types and explorer]] and [[client vs server and replication]]

**`Instance:FindFirstChild(childName)`**: Immediately returns the child object or `nil` if it's not found. It does not error or wait. Use this on the server or when you are certain an object exists.
```lua
-- Check if a character has a "Shield" part
local shield = character:FindFirstChild("Shield")
if shield then
	print("Character has a shield.")
end
```