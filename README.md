local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer

-- Create ScreenGui
local gui = Instance.new("ScreenGui", LocalPlayer:WaitForChild("PlayerGui"))
gui.Name = "MyFeatureUI"

-- Create a Frame for the main window
local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 520, 0, 500)
frame.Position = UDim2.new(0.5, -260, 0.5, -250)
frame.BackgroundColor3 = Color3.fromRGB(0, 0, 139)
frame.BorderSizePixel = 0

-- Make draggable
local function makeDraggable(frame)
    local dragging = false
    local dragStart, startPos
    frame.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            dragStart = input.Position
            startPos = frame.Position
            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then
                    dragging = false
                end
            end)
        end
    end)
    UserInputService.InputChanged:Connect(function(input)
        if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
            local delta = input.Position - dragStart
            frame.Position = UDim2.new(
                startPos.X.Scale,
                startPos.X.Offset + delta.X,
                startPos.Y.Scale,
                startPos.Y.Offset + delta.Y
            )
        end
    end)
end
makeDraggable(frame)

-- Title
local title = Instance.new("TextLabel", frame)
title.Size = UDim2.new(1, 0, 0, 50)
title.BackgroundColor3 = Color3.fromRGB(0, 0, 139)
title.TextColor3 = Color3.new(1, 1, 1)
title.Text = "Invincible Private Killing"
title.Font = Enum.Font.SourceSansBold
title.TextSize = 20

local yOffset = 60

-- Helper to add buttons
local function addButton(text, callback)
    local btn = Instance.new("TextButton", frame)
    btn.Size = UDim2.new(0, 300, 0, 40)
    btn.Position = UDim2.new(0, 20, 0, yOffset)
    btn.Text = text
    btn.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    btn.TextColor3 = Color3.new(1,1,1)
    yOffset = yOffset + 45
    btn.MouseButton1Click:Connect(callback)
    return btn
end

-- Helper to add switch
local function addSwitch(text, callback)
    local switch = Instance.new("TextButton", frame)
    switch.Size = UDim2.new(0, 200, 0, 40)
    switch.Position = UDim2.new(0, 0, 139, yOffset)
    switch.Text = text .. " : OFF"
    switch.BackgroundColor3 = Color3.fromRGB(0, 50, 0)
    switch.TextColor3 = Color3.new(1,1,1)
    yOffset = yOffset + 45
    local state = false
    switch.MouseButton1Click:Connect(function()
        state = not state
        switch.Text = text .. " : " .. (state and "ON" or "OFF")
        callback(state)
    end)
    return {switch = switch, get = function() return state end}
end

-- Helper to add dropdown
local function addDropdown(name, callback)
    local dropdown = {}
    local button = addButton(name, function()
        container.Visible = not container.Visible
    end)
    local container = Instance.new("Frame", frame)
    container.Size = UDim2.new(0, 300, 0, 0)
    container.Position = UDim2.new(0, 20, 0, yOffset)
    container.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
    container.Visible = false
    local listLayout = Instance.new("UIListLayout", container)
    local totalHeight = 0

    function dropdown:Add(itemName)
        local btn = addButton(itemName, function()
            callback(itemName)
            container.Visible = false
        end)
        btn.Parent = container
        totalHeight = totalHeight + 45
        container.Size = UDim2.new(0, 300, 0, totalHeight)
    end

    function dropdown:Clear()
        for _, v in pairs(container:GetChildren()) do
            if v:IsA("TextButton") then v:Destroy() end
        end
        container.Size = UDim2.new(0, 300, 0, 0)
        totalHeight = 0
    end

    function dropdown:GetSelected()
        return dropdown.SelectedItem
    end

    dropdown.Content = container
    return dropdown
end

-- Now create a basic tab-like structure (since no custom library, just a label)
local tabLabel = Instance.new("TextLabel", frame)
tabLabel.Size = UDim2.new(1, 0, 0, 30)
tabLabel.Position = UDim2.new(0, 0, 0, 0)
tabLabel.Text = "Main Tab"
tabLabel.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
tabLabel.TextColor3 = Color3.new(1, 1, 1)
tabLabel.TextSize = 16
tabLabel.Font = Enum.Font.SourceSansBold

-- Example button to test
addButton("Test Button", function()
    print("Button clicked!")
end)

-- Example switch
addSwitch("Test Switch", function(state)
    print("Switch state:", state)
end)

-- Example dropdown
local exampleDropdown = addDropdown("Choose Option", function(selected)
    print("Selected:", selected)
end)
exampleDropdown:Add("Option 1")
exampleDropdown:Add("Option 2")
exampleDropdown:Add("Option 3")
