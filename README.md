## Create Window
```lua
local Warrior = {
    Tabs = {}
}

local UserInputService = game:GetService("UserInputService")

local Objects = game:GetService("ReplicatedStorage"):FindFirstChild("WarriorObjects") or game:GetObjects("rbxassetid://18839887149")[1]:Clone()
local UI = Objects.Warrior:Clone()
UI.Parent = (gethui and gethui()) or (game:GetService("RunService"):IsStudio() and game:GetService("Players").LocalPlayer.PlayerGui or game:GetService("CoreGui"))

local Tab = {}; do
    Tab.__index = Tab
    
    function Tab:NewToggle(name, default, callback)
        local ToggleUI = Objects.Toggle:Clone()
        ToggleUI.TextLabel.Text = name
        
        local currentValue = default
        
        local function Toggle(v)
            currentValue = v
            if v then
                ToggleUI.TextButton.Frame.BackgroundColor3 = Color3.fromHex("#55ff7f")
                ToggleUI.TextButton.Frame.Position = UDim2.new(0.483, 0, 0, 0)
            else
                ToggleUI.TextButton.Frame.BackgroundColor3 = Color3.fromHex("#ff061f")
                ToggleUI.TextButton.Frame.Position = UDim2.new(-0.017, 0, 0, 0)
            end
            
            callback(v)
        end
        
        Toggle(default)
        
        table.insert(self.Elements, ToggleUI)
        
        ToggleUI.TextButton.MouseButton1Click:Connect(function()
            currentValue = not currentValue
            Toggle(currentValue)
        end)
    end
    
    function Tab:NewSlider(name, rangeMin, rangeMax, default, callback)
        local dragging = false
        local range = {rangeMin, rangeMax}
        
        local ui = Objects.Slider:Clone()
        ui.TextLabel.Text = name
        
        table.insert(self.Elements, ui)

        local Update = function(v)
            local percentage = (v - range[1]) / (range[2] - range[1])

            ui.Frame.Frame.Size = UDim2.new(percentage, 0, 1, 0);
            ui.TextLabel.Text = name.." ("..(math.round(v * 10) / 10)..")"
            
            callback(v)
        end

        Update(default)

        UserInputService.InputChanged:Connect(function(input)
            if dragging then
                local mousePos = UserInputService:GetMouseLocation()
                local mouseX, mouseY = mousePos.X, mousePos.Y
                local boundaries0 = ui.Frame.AbsolutePosition.X 
                local boundaries1 = ui.Frame.AbsolutePosition.X + ui.Frame.AbsoluteSize.X
                local at = mouseX - boundaries0
                local goal = boundaries1 - boundaries0
                local percentage = math.clamp(at / goal, 0, 1)
                Update(rangeMin + ((rangeMax - rangeMin) * percentage))
            end
        end)

        ui.Frame.InputBegan:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 then
                dragging = true
                local e; e = input.Changed:Connect(function()
                    if input.UserInputState == Enum.UserInputState.End then
                        dragging = false
                        e:Disconnect()
                    end
                end)
            end
        end)
    end
    
    function Tab:NewDropdown(name, range, default, callback)
        local DropdownUI = Objects.Dropdown:Clone()
        DropdownUI.TextLabel.Text = name
        
        local rangeIndex = 1
        
        local Update = function(v)
            if rangeIndex > #range then
                rangeIndex = 1
            end
            
            DropdownUI.TextButton.Text = range[rangeIndex]
            
            callback(v)
        end
        
        Update(table.find(range, default))
        
        table.insert(self.Elements, DropdownUI)
        
        DropdownUI.TextButton.MouseButton1Click:Connect(function()
            rangeIndex += 1
            Update(rangeIndex)
        end)
    end
end

function Warrior:NewTab(name)
    local tab = setmetatable({}, Tab)
    
    tab.Button = Objects.TabButton:Clone()
    tab.Button.Parent = UI.Frame.Tabs
    tab.Button.Text = name
    tab.Elements = {}
    
    tab.Button.MouseButton1Click:Connect(function()
        for index, value in pairs(UI.Frame.Elements:GetChildren()) do
            if value:IsA("UIListLayout") then continue end
            value.Parent = nil
        end
        
        for index, value in pairs(tab.Elements) do
            value.Parent = UI.Frame.Elements
        end
    end)
    
    return tab, table.insert(Warrior.Tabs, tab)
end

do
    local gui = UI.Frame

    local dragging
    local dragInput
    local dragStart
    local startPos

    local function update(input)
        local delta = input.Position - dragStart
        gui.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end

    gui.Frame.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            dragStart = input.Position
            startPos = gui.Position

            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then
                    dragging = false
                end
            end)
        end
    end)

    gui.Frame.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
            dragInput = input
        end
    end)

    UserInputService.InputChanged:Connect(function(input)
        if input == dragInput and dragging then
            update(input)
        end
    end)
end
```

## Create Slider
```lua
local exampleTab = Warrior:NewTab("Example Tab")
```

## Create Toggle
```lua
exampleTab:NewToggle("Enable Feature", false, function(state)
    print("Feature enabled:", state)
end)
```
## Create Slider
```lua
exampleTab:NewSlider("Adjust Speed", 1, 100, 50, function(value)
    print("Speed set to:", value)
end)
```
## Create Dropdown
```lua
exampleTab:NewDropdown("Select Option", {"Option 1", "Option 2", "Option 3"}, "Option 1", function(selected)
    print("Selected option:", selected)
end)
```

## Put this at the end of the script
```lua
return Warrior
```
