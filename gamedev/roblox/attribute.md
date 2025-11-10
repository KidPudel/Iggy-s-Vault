Custom values/properties on the object

```lua
local part = game.Workspace.Part

part:SetAttribute("Health", 100)

local health = part:GetAttribute("Health")

part:GetAttributeChangedSignal("Health", function()
	if health <= 0 then
		part:Destroy()		
	end
end)

part:SetAttribute("Health", health - 10)
```