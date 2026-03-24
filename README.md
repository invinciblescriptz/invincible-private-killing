-- Full feature-rich GUI with draggable functionality

local player = game.Players.LocalPlayer
local gui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
gui.Name = "MyFeatureUI"

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 650, 0, 500)
frame.Position = UDim2.new(0.5, -325, 0.5, -250)
frame.BackgroundColor3 = Color3.fromRGB(0, 10, 0)
frame.BorderSizePixel = 0

-- Make the frame draggable
do
    local userInputService = game:GetService("UserInputService")
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

    userInputService.InputChanged:Connect(function(input)
        if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
            local delta = input.Position - dragStart
            frame.Position = startPos + UDim2.new(0, delta.X, 0, delta.Y)
        end
    end)
end

-- Title label
local title = Instance.new("TextLabel", frame)
title.Size = UDim2.new(1, 0, 0, 50)
title.BackgroundColor3 = Color3.fromRGB(0, 20, 0)
title.TextColor3 = Color3.new(1, 1, 1)
title.Text = "Invincible Private Killing"
title.Font = Enum.Font.SourceSansBold
title.TextSize = 20

local yOffset = 60

-- Helper functions to create buttons and inputs
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

-- Placeholder for tab and other GUI elements (assuming you have your GUI library)
-- For simplicity, I'll create a main container for 'Kill' tab features
local KillTabFrame = Instance.new("Frame", frame)
KillTabFrame.Size = UDim2.new(1, -40, 1, -70)
KillTabFrame.Position = UDim2.new(0, 20, 0, 70)
KillTabFrame.BackgroundTransparency = 1

-- Function to add label
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

-- Function to add switch
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

-- Function to add dropdown
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
    label.PaddingLeft = 5

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

-- --- START adding features ---

local y = 10 -- vertical offset for features

-- Example: Add label for Pet selection
addLabel(KillTabFrame, "Select damage or durability pet", y)
y = y + 25

-- Dropdown for selecting pet
local petDropdown = addDropdown(KillTabFrame, "Select Pet", function(selected)
    -- Your pet equip logic here
    local petsFolder = game.Players.LocalPlayer.petsFolder
    for _, folder in pairs(petsFolder:GetChildren()) do
        if folder:IsA("Folder") then
            for _, pet in pairs(folder:GetChildren()) do
                game:GetService("ReplicatedStorage").rEvents.equipPetEvent:FireServer("unequipPet", pet)
            end
        end
    end
    task.wait(0.2)
    local petsToEquip = {}
    for _, pet in pairs(game.Players.LocalPlayer.petsFolder.Unique:GetChildren()) do
        if pet.Name == selected then
            table.insert(petsToEquip, pet)
        end
    end
    local maxPets = 8
    local count = math.min(#petsToEquip, maxPets)
    for i = 1, count do
        game:GetService("ReplicatedStorage").rEvents.equipPetEvent:FireServer("equipPet", petsToEquip[i])
        task.wait(0.1)
    end
end)
y = y + 35

-- Add switches
local autoGoodKarmaSwitch = addSwitch(KillTabFrame, "Auto Good Karma", function(state)
    autoGoodKarma = state
    if state then
        task.spawn(function()
            while autoGoodKarma do
                local playerChar = game.Players.LocalPlayer.Character
                local rightHand = playerChar and playerChar:FindFirstChild("RightHand")
                local leftHand = playerChar and playerChar:FindFirstChild("LeftHand")
                if playerChar and rightHand and leftHand then
                    for _, target in ipairs(game.Players:GetPlayers()) do
                        if target ~= game.Players.LocalPlayer then
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
y = y + 40

local autoBadKarmaSwitch = addSwitch(KillTabFrame, "Auto Bad Karma", function(state)
    autoBadKarma = state
    if state then
        task.spawn(function()
            while autoBadKarma do
                local playerChar = game.Players.LocalPlayer.Character
                local rightHand = playerChar and playerChar:FindFirstChild("RightHand")
                local leftHand = playerChar and playerChar:FindFirstChild("LeftHand")
                if playerChar and rightHand and leftHand then
                    for _, target in ipairs(game.Players:GetPlayers()) do
                        if target ~= game.Players.LocalPlayer then
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
y = y + 40

-- Auto whitelist friends
local playerWhitelist = {}
local friendWhitelistActive = false

local whitelistSwitch = addSwitch(KillTabFrame, "Auto Whitelist Friends", function(state)
    friendWhitelistActive = state
    if state then
        for _, player in ipairs(game.Players:GetPlayers()) do
            if player ~= game.Players.LocalPlayer and game.Players.LocalPlayer:IsFriendsWith(player.UserId) then
                playerWhitelist[player.Name] = true
            end
        end
        game.Players.PlayerAdded:Connect(function(p)
            if friendWhitelistActive and p ~= game.Players.LocalPlayer and game.Players.LocalPlayer:IsFriendsWith(p.UserId) then
                playerWhitelist[p.Name] = true
            end
        end)
    else
        for name in pairs(playerWhitelist) do
            local p = game.Players:FindFirstChild(name)
            if p and game.Players.LocalPlayer:IsFriendsWith(p.UserId) then
                playerWhitelist[name] = nil
            end
        end
    end
end)
y = y + 40

-- Whitelist and UnWhitelist text boxes
local whitelistInput = createInput("Whitelist", y)
whitelistInput.FocusLost:Connect(function()
    local target = game.Players:FindFirstChild(whitelistInput.Text)
    if target then
        playerWhitelist[target.Name] = true
    end
end)
y = y + 45

local unWhitelistInput = createInput("UnWhitelist", y)
unWhitelistInput.FocusLost:Connect(function()
    local target = game.Players:FindFirstChild(unWhitelistInput.Text)
    if target then
        playerWhitelist[target.Name] = nil
    end
end)
y = y + 45

-- Auto Kill
local autoKillSwitch = addSwitch(KillTabFrame, "Auto Kill", function(state)
    autoKill = state
    if state then
        task.spawn(function()
            while autoKill do
                local character = game.Players.LocalPlayer.Character or game.Players.LocalPlayer.CharacterAdded:Wait()
                local rightHand = character:FindFirstChild("RightHand")
                local leftHand = character:FindFirstChild("LeftHand")
                local punch = game.Players.LocalPlayer.Backpack:FindFirstChild("Punch")
                if punch and not character:FindFirstChild("Punch") then
                    punch.Parent = character
                end
                if rightHand and leftHand then
                    for _, target in ipairs(game.Players:GetPlayers()) do
                        if target ~= game.Players.LocalPlayer and not playerWhitelist[target.Name] then
                            local tChar = target.Character
                            local rootPart = tChar and tChar:FindFirstChild("HumanoidRootPart")
                            if rootPart then
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
y = y + 40

-- Target selection dropdown
local targetPlayerNames = {}
local selectedTarget = nil

local function updateTargetDropdown()
    -- You can implement dynamic dropdown updates if needed
end

local targetDropdown = addDropdown(KillTabFrame, "Select Target", function(displayName)
    for _, player in ipairs(game.Players:GetPlayers()) do
        if player.DisplayName == displayName then
            selectedTarget = player.Name
            break
        end
    end
end)

-- Populate initial players
for _, player in ipairs(game.Players:GetPlayers()) do
    if player ~= game.Players.LocalPlayer then
        targetDropdown.Add(player.DisplayName)
        table.insert(targetPlayerNames, player.Name)
    end
end

game.Players.PlayerAdded:Connect(function(player)
    if player ~= game.Players.LocalPlayer then
        targetDropdown.Add(player.DisplayName)
        table.insert(targetPlayerNames, player.Name)
    end
end)

game.Players.PlayerRemoving:Connect(function(player)
    -- Remove from list if needed
    -- (You can enhance this to remove from dropdown)
end)

-- Start kill target toggle
local killTarget = false
local function toggleKillTarget(state)
    killTarget = state
    if state then
        task.spawn(function()
            while killTarget do
                local character = game.Players.LocalPlayer.Character or game.Players.LocalPlayer.CharacterAdded:Wait()
                local rightHand = character:FindFirstChild("RightHand")
                local leftHand = character:FindFirstChild("LeftHand")
                local punch = game.Players.LocalPlayer.Backpack:FindFirstChild("Punch")
                if punch and not character:FindFirstChild("Punch") then
                    punch.Parent = character
                end
                if rightHand and leftHand then
                    for _, name in ipairs(targetPlayerNames) do
                        local target = game.Players:FindFirstChild(name)
                        if target and target ~= game.Players.LocalPlayer and target.Character then
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
end

local toggleKillButton = createButton("Start Kill Target", y)
toggleKillButton.MouseButton1Click:Connect(function()
    toggleKillTarget(not killTarget)
end)
y = y + 45

-- View Player (Spy) dropdown
local targetViewPlayer = nil
local spying = false

local function followPlayer(targetPlayer)
    local myChar = game.Players.LocalPlayer.Character
    local targetChar = targetPlayer.Character
    if not myChar or not targetChar then return end
    local myHRP = myChar:FindFirstChild("HumanoidRootPart")
    local targetHRP = targetChar:FindFirstChild("HumanoidRootPart")
    if myHRP and targetHRP then
        local followPos = targetHRP.CFrame.Position - (targetHRP.CFrame.LookVector * 3)
        myHRP.CFrame = CFrame.new(followPos, targetHRP.Position)
    end
end

local viewDropdown = addDropdown(KillTabFrame, "Select View Target", function(displayName)
    for _, player in ipairs(game.Players:GetPlayers()) do
        if player.DisplayName == displayName then
            targetViewPlayer = player
            break
        end
    end
end)

-- Populate initial
for _, player in ipairs(game.Players:GetPlayers()) do
    if player ~= game.Players.LocalPlayer then
        viewDropdown.Add(player.DisplayName)
    end
end

game.Players.PlayerAdded:Connect(function(player)
    if player ~= game.Players.LocalPlayer then
        viewDropdown.Add(player.DisplayName)
    end
end)

game.Players.PlayerRemoving:Connect(function(player)
    -- Optional: handle removal
end)

local function toggleSpy(state)
    spying = state
    if not spying then
        workspace.CurrentCamera.CameraSubject = game.Players.LocalPlayer.Character and game.Players.LocalPlayer.Character:FindFirstChild("Humanoid") or game.Players.LocalPlayer
        return
    end
    task.spawn(function()
        while spying do
            if targetViewPlayer then
                local humanoid = targetViewPlayer.Character and targetViewPlayer.Character:FindFirstChild("Humanoid")
                if humanoid then
                    workspace.CurrentCamera.CameraSubject = humanoid
                end
            end
            task.wait(0.1)
        end
    end)
end

local viewSwitch = addSwitch(KillTabFrame, "View Player", toggleSpy)

-- Remove Punch Anim button
local removePunchButton = createButton("Remove Punch Anim", y)
removePunchButton.MouseButton1Click:Connect(function()
    -- Your existing code for removing punch animations
    -- (Omitted for brevity; keep your existing implementation here)
end)
y = y + 45

-- Auto Equip Punch
local autoEquipPunchSwitch = addSwitch(KillTabFrame, "Auto Equip Punch", function(state)
    autoEquipPunch = state
    task.spawn(function()
        while autoEquipPunch do
            local punch = game.Players.LocalPlayer.Backpack:FindFirstChild("Punch")
            if punch then
                punch.Parent = game.Players.LocalPlayer.Character
            end
            task.wait(0.1)
        end
    end)
end)
y = y + 40

-- Auto Punch without animation
local autoPunchNoAnimSwitch = addSwitch(KillTabFrame, "Auto Punch [without animation]", function(state)
    autoPunchNoAnim = state
    task.spawn(function()
        while autoPunchNoAnim do
            local punch = game.Players.LocalPlayer.Backpack:FindFirstChild("Punch") or game.Players.LocalPlayer.Character and game.Players.LocalPlayer.Character:FindFirstChild("Punch")
            if punch then
                if punch.Parent ~= game.Players.LocalPlayer.Character then
                    punch.Parent = game.Players.LocalPlayer.Character
                end
                game.Players.LocalPlayer.muscleEvent:FireServer("punch", "rightHand")
                game.Players.LocalPlayer.muscleEvent:FireServer("punch", "leftHand")
            else
                autoPunchNoAnim = false
            end
            task.wait(0.01)
        end
    end)
end)
y = y + 40

-- Auto Punch (with animation)
local autoPunchSwitch = addSwitch(KillTabFrame, "Auto Punch", function(state)
    _G.fastHitActive = state
    if state then
        task.spawn(function()
            while _G.fastHitActive do
                local punch = game.Players.LocalPlayer.Backpack:FindFirstChild("Punch")
                if punch then
                    punch.Parent = game.Players.LocalPlayer.Character
                    if punch:FindFirstChild("attackTime") then
                        punch.attackTime.Value = 0
                    end
                end
                task.wait(0.1)
            end
        end)
        task.spawn(function()
            while _G.fastHitActive do
                local punch = game.Players.LocalPlayer.Character and game.Players.LocalPlayer.Character:FindFirstChild("Punch")
                if punch then
                    punch:Activate()
                end
                task.wait(0.1)
            end
        end)
    else
        local punch = game.Players.LocalPlayer.Character and game.Players.LocalPlayer.Character:FindFirstChild("Punch")
        if punch then
            punch.Parent = game.Players.LocalPlayer.Backpack
        end
    end
end)
y = y + 40

-- Fast Punch
local fastPunchSwitch = addSwitch(KillTabFrame, "Fast punch", function(state)
    _G.autoPunchActive = state
    if state then
        task.spawn(function()
            while _G.autoPunchActive do
                local punch = game.Players.LocalPlayer.Backpack:FindFirstChild("Punch")
                if punch then
                    punch.Parent = game.Players.LocalPlayer.Character
                    if punch:FindFirstChild("attackTime") then
                        punch.attackTime.Value = 0
                    end
                end
                task.wait()
            end
        end)
        task.spawn(function()
            while _G.autoPunchActive do
                local punch = game.Players.LocalPlayer.Character and game.Players.LocalPlayer.Character:FindFirstChild("Punch")
                if punch then
                    punch:Activate()
                end
                task.wait()
            end
        end)
    else
        local punch = game.Players.LocalPlayer.Character and game.Players.LocalPlayer.Character:FindFirstChild("Punch")
        if punch then
            punch.Parent = game.Players.LocalPlayer.Backpack
        end
    end
end)
y = y + 40

-- God Mode toggle
local godModeToggle = false
local godModeSwitch = addSwitch(KillTabFrame, "Good mode", function(state)
    godModeToggle = state
    if state then
        task.spawn(function()
            while godModeToggle do
                game:GetService("ReplicatedStorage").rEvents.brawlEvent:FireServer("joinBrawl")
                task.wait()
            end
        end)
    end
end)
y = y + 40

-- Teleport / Follow System
local following = false
local followTarget = nil

-- Function to follow a player
local function followPlayer(targetPlayer)
    local myChar = game.Players.LocalPlayer.Character
    local targetChar = targetPlayer.Character
    if not (myChar and targetChar) then return end
    local myHRP = myChar:FindFirstChild("HumanoidRootPart")
    local targetHRP = targetChar:FindFirstChild("HumanoidRootPart")
    if myHRP and targetHRP then
        local followPos = targetHRP.CFrame.Position - (targetHRP.CFrame.LookVector * 3)
        myHRP.CFrame = CFrame.new(followPos, targetHRP.Position)
    end
end

local followDropdown = addDropdown(KillTabFrame, "Teleport player", function(selectedDisplayName)
    for _, plr in ipairs(game.Players:GetPlayers()) do
        if plr.DisplayName == selectedDisplayName then
            followTarget = plr
            following = true
            -- Immediate follow
            followPlayer(plr)
            print("✅ Started following:", plr.Name)
            break
        end
    end
end)

-- Populate initial players
for _, player in ipairs(game.Players:GetPlayers()) do
    if player ~= game.Players.LocalPlayer then
        followDropdown.Add(player.DisplayName)
    end
end

game.Players.PlayerAdded:Connect(function(player)
    if player ~= game.Players.LocalPlayer then
        followDropdown.Add(player.DisplayName)
    end
end)

game.Players.PlayerRemoving:Connect(function(player)
    -- Handle removal if needed
    if followTarget and followTarget.Name == player.Name then
        followTarget = nil
        following = false
    end
end)

-- Button to stop following
local stopFollowBtn = createButton("Dejar de Seguir", y)
stopFollowBtn.MouseButton1Click:Connect(function()
    following = false
    followTarget = nil
    print("⛔ Stopped following")
end)
y = y + 45

-- Loop for auto-follow
task.spawn(function()
    while true do
        if following and followTarget then
            followPlayer(followTarget)
        end
        task.wait(0.01)
    end
end)

-- Re-follow after respawn
game.Players.LocalPlayer.CharacterAdded:Connect(function()
    task.wait(1)
    if following and followTarget then
        followPlayer(followTarget)
    end
end)

-- Auto Slams (auto ground slam)
local autoSlam = false
local slamSwitch = addSwitch(KillTabFrame, "auto slams", function(state)
    autoSlam = state
    if state then
        task.spawn(function()
            while autoSlam do
                local player = game.Players.LocalPlayer
                local groundSlam = player.Backpack:FindFirstChild("Ground Slam") or (player.Character and player.Character:FindFirstChild("Ground Slam"))
                if groundSlam then
                    if groundSlam.Parent == player.Backpack then
                        groundSlam.Parent = player.Character
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
end)
y = y + 40

-- "Combo NaN" button (execute raw URL scripts)
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
                if not loadSuccess then
                    warn("[Pegar Muerto] Error ejecutando raw:", url, err)
                end
            else
                warn("[Pegar Muerto] No se pudo cargar:", url)
            end
        end)
    end
end

local function createButton(text, yPos)
    local btn = Instance.new("TextButton", frame)
    btn.Size = UDim2.new(0, 300, 0, 40)
    btn.Position = UDim2.new(0, 20, 0, yPos)
    btn.Text = text
    btn.BackgroundColor3 = Color3.fromRGB(0, 50, 0)
    btn.TextColor3 = Color3.new(1, 1, 1)
    return btn
end

local crackBtn = createButton("Touch Me!", y)
crackBtn.MouseButton1Click:Connect(function()
    runCracks()
end)
y = y + 45

-- Change Time Dropdown
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
local timeDropdown = addDropdown(KillTabFrame, "change time", function(selection)
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
end)

-- Populate dropdown options
for _, option in ipairs(timeOptions) do
    timeDropdown:Add(option)
end
