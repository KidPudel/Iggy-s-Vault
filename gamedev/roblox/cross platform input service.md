Use `ContextActionService` for cross-platform input:

```lua
local ContextActionService = game:GetService("ContextActionService")

local function handleAction(actionName, inputState, inputObject)
    if inputState == Enum.UserInputState.Begin then
        print("Action pressed")
        -- Do your action here
    end
end

-- Bind to keyboard AND create mobile button
ContextActionService:BindAction(
    "MyAction",           -- Action name
    handleAction,         -- Function to call
    true,                 -- Create mobile button (true/false)
    Enum.KeyCode.E        -- PC keybind
)
```

This automatically:

- Works with keyboard on PC
- Creates a touchscreen button on mobile
- Handles controller input

To customize the mobile button:

```lua
ContextActionService:SetImage("MyAction", "rbxassetid://YOUR_ICON_ID")
ContextActionService:SetPosition("MyAction", UDim2.new(1, -70, 0, 10))
```

Or if you want manual mobile controls, detect touch:

```lua
UserInputService.TouchTap:Connect(function(touchPositions, gameProcessed)
    print("Screen tapped")
end)
```

ContextActionService is the standard approach for games that need to work on all platforms.