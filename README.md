-- Services
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local Lighting = game:GetService("Lighting")
local Workspace = game:GetService("Workspace")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer

-- Create Main Screen GUI
local gui = Instance.new("ScreenGui", LocalPlayer:WaitForChild("PlayerGui"))
gui.Name = "MyFeatureUI"

local mainFrame = Instance.new("Frame", gui)
mainFrame.Size = UDim2.new(0, 600, 0, 700)
mainFrame.Position = UDim2.new(0.5, -300, 0.5, -350)
mainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
mainFrame.BorderSizePixel = 0

-- Make GUI draggable
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

makeDraggable(mainFrame)

-- UI Templates
local function createSection(title, yOffset)
    local section = Instance.new("Frame", mainFrame)
    section.Size = UDim2.new(1, -20, 0, 240)
    section.Position = UDim2.new(0, 10, 0, yOffset)
    section.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    section.BorderSizePixel = 0
    
    local label = Instance.new("TextLabel", section)
    label.Text = title
    label.Size = UDim2.new(1, 0, 0, 24)
    label.BackgroundTransparency = 1
    label.TextColor3 = Color3.fromRGB(255, 255, 255)
    label.Font = Enum.Font.Merriweather
    label.TextSize = 18
    
    return section
end

-- Sections
local petsSection = createSection("Pets", 10)
local karmaSection = createSection("Karma", 260)
local killSection = createSection("Kill System", 510)
local followSection = createSection("Follow System", 760)
local timeSection = createSection("Lighting Time", 1010)

-- ========================
-- Pets Dropdown
local petsDropdown = Instance.new("TextButton", petsSection)
petsDropdown.Size = UDim2.new(0, 200, 0, 30)
petsDropdown.Position = UDim2.new(0, 10, 0, 30)
petsDropdown.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
petsDropdown.Text = "Select Pet"
petsDropdown.TextColor3 = Color3.fromRGB(255, 255, 255)
petsDropdown.Font = Enum.Font.Merriweather
petsDropdown.TextSize = 14

local petsOptionsFrame = Instance.new("Frame", petsSection)
petsOptionsFrame.Size = UDim2.new(0, 200, 0, 0)
petsOptionsFrame.Position = UDim2.new(0, 10, 0, 60)
petsOptionsFrame.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
petsOptionsFrame.BorderSizePixel = 1
petsOptionsFrame.Visible = false

local petsOptions = {}

local function addPetOption(text)
    local btn = Instance.new("TextButton", petsOptionsFrame)
    btn.Size = UDim2.new(1, 0, 0, 30)
    btn.Text = text
    btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    btn.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
    btn.MouseButton1Click:Connect(function()
        -- Equip pet
        local petsFolder = LocalPlayer:WaitForChild("petsFolder")
        for _, folder in pairs(petsFolder:GetChildren()) do
            if folder:IsA("Folder") then
                for _, pet in pairs(folder:GetChildren()) do
                    game:GetService("ReplicatedStorage").rEvents.equipPetEvent:FireServer("unequipPet", pet)
                end
            end
        end
        task.wait(0.2)
        for _, pet in pairs(petsFolder:FindFirstChild("Unique"):GetChildren()) do
            if pet.Name == text then
                game:GetService("ReplicatedStorage").rEvents.equipPetEvent:FireServer("equipPet", pet)
            end
        end
        petsOptionsFrame.Visible = false
        petsDropdown.Text = text
    end)
    table.insert(petsOptions, btn)
end

petsDropdown.MouseButton1Click:Connect(function()
    petsOptionsFrame.Visible = not petsOptionsFrame.Visible
    petsOptionsFrame.Size = UDim2.new(0, 200, 0, #petsOptions * 30)
end)

-- Add some default pets for demo
local function populatePets()
    local petNames = {"Wild Wizard", "Mighty Monster"}
    for _, name in ipairs(petNames) do
        addPetOption(name)
    end
end
populatePets()

-- ========================
-- Karma toggles
local autoGoodKarma = false
local autoBadKarma = false
local function createSwitch(parent, text, callback)
    local switchBtn = Instance.new("TextButton", parent)
    switchBtn.Size = UDim2.new(0, 200, 0, 30)
    switchBtn.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
    switchBtn.Text = text .. ": OFF"
    switchBtn.TextColor3 = Color3.fromRGB(255,255,255)
    switchBtn.Font = Enum.Font.Merriweather
    switchBtn.TextSize = 14
    
    local active = false
    switchBtn.MouseButton1Click:Connect(function()
        active = not active
        switchBtn.Text = text .. ": " .. (active and "ON" or "OFF")
        callback(active)
    end)
    return switchBtn
end

-- Auto Good Karma
createSwitch(karmaSection, "Auto Good Karma", function(enabled)
    autoGoodKarma = enabled
    if enabled then
        task.spawn(function()
            while autoGoodKarma do
                local character = LocalPlayer.Character
                local rightHand = character and character:FindFirstChild("RightHand")
                local leftHand = character and character:FindFirstChild("LeftHand")
                if character and rightHand and leftHand then
                    for _, target in ipairs(Players:GetPlayers()) do
                        if target ~= LocalPlayer then
                            local evilKarma = target:FindFirstChild("evilKarma")
                            local goodKarma = target:FindFirstChild("goodKarma")
                            if evilKarma and goodKarma and evilKarma:IsA("IntValue") and goodKarma:IsA("IntValue") then
                                if evilKarma.Value > goodKarma.Value then
                                    local rootPart = target.Character and target.Character:FindFirstChild("HumanoidRootPart")
                                    if rootPart then
                                        firetouchinterest(rightHand, rootPart, 1)
                                        firetouchinterest(leftHand, rootPart, 1)
                                        firetouchinterest(rightHand, rootPart, 0)
                                        firetouchinterest(leftHand, rootPart, 0)
                                    end
                                end
                            end
                        end
                    end
                end
                task.wait(0.01)
            end
        end)
    end
end)

-- Auto Bad Karma
createSwitch(karmaSection, "Auto Bad Karma", function(enabled)
    autoBadKarma = enabled
    if enabled then
        task.spawn(function()
            while autoBadKarma do
                local character = LocalPlayer.Character
                local rightHand = character and character:FindFirstChild("RightHand")
                local leftHand = character and character:FindFirstChild("LeftHand")
                if character and rightHand and leftHand then
                    for _, target in ipairs(Players:GetPlayers()) do
                        if target ~= LocalPlayer then
                            local evilKarma = target:FindFirstChild("evilKarma")
                            local goodKarma = target:FindFirstChild("goodKarma")
                            if evilKarma and goodKarma and evilKarma:IsA("IntValue") and goodKarma:IsA("IntValue") then
                                if goodKarma.Value > evilKarma.Value then
                                    local rootPart = target.Character and target.Character:FindFirstChild("HumanoidRootPart")
                                    if rootPart then
                                        firetouchinterest(rightHand, rootPart, 1)
                                        firetouchinterest(leftHand, rootPart, 1)
                                        firetouchinterest(rightHand, rootPart, 0)
                                        firetouchinterest(leftHand, rootPart, 0)
                                    end
                                end
                            end
                        end
                    end
                end
                task.wait(0.01)
            end
        end)
    end
end)

-- ========================
-- Whitelist Friends
local playerWhitelist = {}
local function createWhitelistSwitch(parent, text)
    local btn = createSwitch(parent, text, function(active)
        if active then
            for _, player in ipairs(Players:GetPlayers()) do
                if player ~= LocalPlayer and LocalPlayer:IsFriendsWith(player.UserId) then
                    playerWhitelist[player.Name] = true
                end
            end
            Players.PlayerAdded:Connect(function(player)
                if LocalPlayer:IsFriendsWith(player.UserId) then
                    playerWhitelist[player.Name] = true
                end
            end)
        else
            for name in pairs(playerWhitelist) do
                if Players:FindFirstChild(name) then
                    playerWhitelist[name] = nil
                end
            end
        end
    end)
    return btn
end

createWhitelistSwitch(karmaSection, "Auto Whitelist Friends")

-- Whitelist / UnWhitelist TextBoxes
local function createTextBox(parent, label, callback)
    local labelText = Instance.new("TextLabel", parent)
    labelText.Size = UDim2.new(1, -10, 0, 20)
    labelText.Position = UDim2.new(0, 5, 0, 0)
    labelText.Text = label
    labelText.TextColor3 = Color3.fromRGB(255,255,255)
    labelText.BackgroundTransparency = 1
    labelText.Font = Enum.Font.Merriweather
    labelText.TextSize = 14

    local textbox = Instance.new("TextBox", parent)
    textbox.Size = UDim2.new(1, -10, 0, 25)
    textbox.Position = UDim2.new(0, 5, 0, 25)
    textbox.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
    textbox.TextColor3 = Color3.fromRGB(255,255,255)
    textbox.Font = Enum.Font.Merriweather
    textbox.TextSize = 14
    textbox.PlaceholderText = "Enter Player Name"

    textbox.FocusLost:Connect(function()
        callback(textbox.Text)
        textbox.Text = ""
    end)
end

createTextBox(karmaSection, "Whitelist Player", function(name)
    local target = Players:FindFirstChild(name)
    if target then
        playerWhitelist[target.Name] = true
    end
end)

createTextBox(karmaSection, "UnWhitelist Player", function(name)
    local target = Players:FindFirstChild(name)
    if target then
        playerWhitelist[target.Name] = nil
    end
end)

-- ========================
-- Kill System
local autoKill = false
local targetPlayerNames = {}
local selectedTarget = nil

local killSwitch = createSwitch(killSection, "Auto Kill", function(enabled)
    autoKill = enabled
    if enabled then
        task.spawn(function()
            while autoKill do
                local character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
                local rightHand = character:FindFirstChild("RightHand")
                local leftHand = character:FindFirstChild("LeftHand")
                local punch = LocalPlayer.Backpack:FindFirstChild("Punch")
                if punch and not character:FindFirstChild("Punch") then
                    punch.Parent = character
                end
                if rightHand and leftHand then
                    for _, name in ipairs(targetPlayerNames) do
                        local target = Players:FindFirstChild(name)
                        if target and target ~= LocalPlayer and target.Character then
                            local rootPart = target.Character:FindFirstChild("HumanoidRootPart")
                            local humanoid = target.Character:FindFirstChild("Humanoid")
                            if rootPart and humanoid and humanoid.Health > 0 then
                                pcall(function()
                                    firetouchinterest(rightHand, rootPart, 1)
                                    firetouchinterest(leftHand, rootPart, 1)
                                    firetouchinterest(rightHand, rootPart, 0)
                                    firetouchinterest(leftHand, rootPart, 0)
                                end)
                            end
                        end
                    end
                end
                task.wait(0.05)
            end
        end)
    end
end)

-- Dropdown for selecting target players
local function createDropdown(parent, label, callback)
    local frame = Instance.new("Frame", parent)
    frame.Size = UDim2.new(0, 220, 0, 30)
    frame.Position = UDim2.new(0, 10, 0, 30)
    frame.BackgroundColor3 = Color3.fromRGB(50, 50, 50)

    local lbl = Instance.new("TextLabel", frame)
    lbl.Text = label
    lbl.Size = UDim2.new(0.4, 0, 1, 0)
    lbl.BackgroundTransparency = 1
    lbl.TextColor3 = Color3.fromRGB(255, 255, 255)
    lbl.Font = Enum.Font.Merriweather
    lbl.TextSize = 14

    local dropdownBtn = Instance.new("TextButton", frame)
    dropdownBtn.Size = UDim2.new(0.6, 0, 1, 0)
    dropdownBtn.Position = UDim2.new(0.4, 0, 0, 0)
    dropdownBtn.Text = "Select"
    dropdownBtn.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
    dropdownBtn.TextColor3 = Color3.fromRGB(255, 255, 255)

    local optionsFrame = Instance.new("Frame", parent)
    optionsFrame.Size = UDim2.new(0, 220, 0, 0)
    optionsFrame.Position = UDim2.new(0, 10, 0, 70)
    optionsFrame.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    optionsFrame.BorderSizePixel = 1
    optionsFrame.Visible = false

    local options = {}

    local function addOption(text)
        local btn = Instance.new("TextButton", optionsFrame)
        btn.Size = UDim2.new(1, 0, 0, 30)
        btn.Text = text
        btn.TextColor3 = Color3.fromRGB(255, 255, 255)
        btn.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
        btn.MouseButton1Click:Connect(function()
            callback(text)
            dropdownBtn.Text = text
            optionsFrame.Visible = false
        end)
        table.insert(options, btn)
    end

    dropdownBtn.MouseButton1Click:Connect(function()
        optionsFrame.Visible = not optionsFrame.Visible
        optionsFrame.Size = UDim2.new(0, 220, 0, #options * 30)
    end)

    return {
        Add = addOption,
        Clear = function()
            for _, btn in pairs(options) do
                btn:Destroy()
            end
            options = {}
            optionsFrame.Size = UDim2.new(0, 220, 0, 0)
            optionsFrame.Visible = false
        end,
        AddOption = addOption,
        SetCallback = callback
    }
end

local targetDropdown = createDropdown(killSection, "Select Target", function(displayName)
    for _, player in ipairs(Players:GetPlayers()) do
        if player.DisplayName == displayName then
            if not table.find(targetPlayerNames, player.Name) then
                table.insert(targetPlayerNames, player.Name)
            end
            selectedTarget = player.Name
            break
        end
    end
end)

-- Initialize with current players
for _, player in ipairs(Players:GetPlayers()) do
    if player ~= LocalPlayer then
        targetDropdown:Add(player.DisplayName)
    end
end

-- Player joins
Players.PlayerAdded:Connect(function(player)
    if player ~= LocalPlayer then
        targetDropdown:Add(player.DisplayName)
    end
end)

-- Player leaves
Players.PlayerRemoving:Connect(function(player)
    -- Remove from list
    if table.find(targetPlayerNames, player.Name) then
        for i, v in ipairs(targetPlayerNames) do
            if v == player.Name then
                table.remove(targetPlayerNames, i)
                break
            end
        end
    end
    -- Refresh dropdown
    targetDropdown:Clear()
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer then
            targetDropdown:Add(plr.DisplayName)
        end
    end
    -- Stop following if target left
    if selectedTarget == player.Name then
        selectedTarget = nil
    end
end)

-- Button to remove selected target
local removeTargetBtn = Instance.new("TextButton", killSection)
removeTargetBtn.Size = UDim2.new(0, 200, 0, 30)
removeTargetBtn.Position = UDim2.new(0, 10, 0, 120)
removeTargetBtn.Text = "Remove Selected Target"
removeTargetBtn.TextColor3 = Color3.fromRGB(255,255,255)
removeTargetBtn.BackgroundColor3 = Color3.fromRGB(150, 0, 0)
removeTargetBtn.MouseButton1Click:Connect(function()
    if selectedTarget then
        for i, v in ipairs(targetPlayerNames) do
            if v == selectedTarget then
                table.remove(targetPlayerNames, i)
                break
            end
        end
        selectedTarget = nil
    end
end)

-- Start Kill Button
local startKillSwitch = createSwitch(killSection, "Start Kill", function(enabled)
    autoKill = enabled
    if enabled then
        task.spawn(function()
            while autoKill do
                local character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
                local rightHand = character:FindFirstChild("RightHand")
                local leftHand = character:FindFirstChild("LeftHand")
                local punch = LocalPlayer.Backpack:FindFirstChild("Punch")
                if punch and not character:FindFirstChild("Punch") then
                    punch.Parent = character
                end
                if rightHand and leftHand then
                    for _, name in ipairs(targetPlayerNames) do
                        local target = Players:FindFirstChild(name)
                        if target and target ~= LocalPlayer and target.Character then
                            local rootPart = target.Character:FindFirstChild("HumanoidRootPart")
                            local humanoid = target.Character:FindFirstChild("Humanoid")
                            if rootPart and humanoid and humanoid.Health > 0 then
                                pcall(function()
                                    firetouchinterest(rightHand, rootPart, 1)
                                    firetouchinterest(leftHand, rootPart, 1)
                                    firetouchinterest(rightHand, rootPart, 0)
                                    firetouchinterest(leftHand, rootPart, 0)
                                end)
                            end
                        end
                    end
                end
                task.wait(0.05)
            end
        end)
    end
end)

-- ========================
-- View Player (Follow) System
local followTargetName = nil
local following = false

local function followPlayer(targetPlayer)
    local myChar = LocalPlayer.Character
    local targetChar = targetPlayer.Character
    if not (myChar and targetChar) then return end
    local myHRP = myChar:FindFirstChild("HumanoidRootPart")
    local targetHRP = targetChar:FindFirstChild("HumanoidRootPart")
    if myHRP and targetHRP then
        local followPos = targetHRP.Position - (targetHRP.CFrame.LookVector * 3)
        myHRP.CFrame = CFrame.new(followPos, targetHRP.Position)
    end
end

local viewDropdown = createDropdown(killSection, "Select View Target", function(displayName)
    for _, player in ipairs(Players:GetPlayers()) do
        if player.DisplayName == displayName then
            followTargetName = player.Name
            local target = player
            following = true
            print("✅ Following: " .. player.Name)
            followPlayer(target)
            break
        end
    end
end)

-- Fill dropdown with current players
for _, player in ipairs(Players:GetPlayers()) do
    if player ~= LocalPlayer then
        viewDropdown:Add(player.DisplayName)
    end
end

-- Player join/leave updates
Players.PlayerAdded:Connect(function(player)
    if player ~= LocalPlayer then
        viewDropdown:Add(player.DisplayName)
    end
end)

Players.PlayerRemoving:Connect(function(player)
    -- Refresh dropdown
    viewDropdown:Clear()
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer then
            viewDropdown:Add(plr.DisplayName)
        end
    end
    -- Stop following if target left
    if followTargetName and player.Name == followTargetName then
        followTargetName = nil
        following = false
    end
end)

-- Button to stop following
local stopFollowBtn = Instance.new("TextButton", killSection)
stopFollowBtn.Size = UDim2.new(0, 200, 0, 30)
stopFollowBtn.Position = UDim2.new(0, 10, 0, 160)
stopFollowBtn.Text = "Stop Following"
stopFollowBtn.TextColor3 = Color3.fromRGB(255,255,255)
stopFollowBtn.BackgroundColor3 = Color3.fromRGB(150, 0, 0)
stopFollowBtn.MouseButton1Click:Connect(function()
    following = false
    followTargetName = nil
    print("⛔ Stopped following")
end)

-- Auto Follow Loop
task.spawn(function()
    while true do
        task.wait(0.01)
        if following and followTargetName then
            local targetPlayer = Players:FindFirstChild(followTargetName)
            if targetPlayer then
                followPlayer(targetPlayer)
            else
                following = false
                followTargetName = nil
            end
        end
    end
end)

-- ========================
-- Lighting Time Control
local timeOptions = {
    "Morning",
    "Noon",
    "Afternoon",
    "Sunset",
    "Night",
    "Midnight",
    "Dawn",
    "Early Morning"
}

local function setLighting(selection)
    -- Reset
    Lighting.Brightness = 2
    Lighting.FogEnd = 100000
    Lighting.Ambient = Color3.fromRGB(127, 127, 127)
    if selection == "Morning" then
        Lighting.ClockTime = 6
        Lighting.Brightness = 2
        Lighting.Ambient = Color3.fromRGB(200, 200, 255)
    elseif selection == "Noon" then
        Lighting.ClockTime = 12
        Lighting.Brightness = 3
        Lighting.Ambient = Color3.fromRGB(255, 255, 255)
    elseif selection == "Afternoon" then
        Lighting.ClockTime = 16
        Lighting.Brightness = 2.5
        Lighting.Ambient = Color3.fromRGB(255, 220, 180)
    elseif selection == "Sunset" then
        Lighting.ClockTime = 18
        Lighting.Brightness = 2
        Lighting.Ambient = Color3.fromRGB(255, 150, 100)
        Lighting.FogEnd = 500
    elseif selection == "Night" then
        Lighting.ClockTime = 20
        Lighting.Brightness = 1.5
        Lighting.Ambient = Color3.fromRGB(100, 100, 150)
        Lighting.FogEnd = 800
    elseif selection == "Midnight" then
        Lighting.ClockTime = 0
        Lighting.Brightness = 1
        Lighting.Ambient = Color3.fromRGB(50, 50, 100)
        Lighting.FogEnd = 400
    elseif selection == "Dawn" then
        Lighting.ClockTime = 4
        Lighting.Brightness = 1.8
        Lighting.Ambient = Color3.fromRGB(180, 180, 220)
    elseif selection == "Early Morning" then
        Lighting.ClockTime = 2
        Lighting.Brightness = 1.2
        Lighting.Ambient = Color3.fromRGB(100, 120, 180)
    end
end

local lightingDropdown = createDropdown(timeSection, "Change Time", function(selection)
    setLighting(selection)
end)

-- Add options to lighting dropdown
for _, option in ipairs(timeOptions) do
    lightingDropdown:Add(option)
end
