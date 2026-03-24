-- Your existing GUI creation code (assuming it's already set up)
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local Lighting = game:GetService("Lighting")
local Workspace = game:GetService("Workspace")
local LocalPlayer = Players.LocalPlayer

local gui = Instance.new("ScreenGui", LocalPlayer:WaitForChild("PlayerGui"))
gui.Name = "MyFeatureUI"

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 520, 0, 500)
frame.Position = UDim2.new(0.5, -260, 0.5, -250)
frame.BackgroundColor3 = Color3.fromRGB(0, 10, 0)
frame.BorderSizePixel = 0

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

local title = Instance.new("TextLabel", frame)
title.Size = UDim2.new(1, 0, 0, 50)
title.BackgroundColor3 = Color3.fromRGB(0, 20, 0)
title.TextColor3 = Color3.new(1, 1, 1)
title.Text = "Invincible Private Killing"
title.Font = Enum.Font.SourceSansBold
title.TextSize = 20
local yOffset = 60

-- Function helpers
local function addButton(text, callback)
    local btn = Instance.new("TextButton", frame)
    btn.Size = UDim2.new(0, 300, 0, 40)
    btn.Position = UDim2.new(0, 20, 0, yOffset)
    btn.Text = text
    btn.BackgroundColor3 = Color3.fromRGB(0, 50, 0)
    btn.TextColor3 = Color3.new(1,1,1)
    yOffset = yOffset + 45
    btn.MouseButton1Click:Connect(callback)
    return btn
end

local function addSwitch(text, callback)
    local switch = Instance.new("TextButton", frame)
    switch.Size = UDim2.new(0, 200, 0, 40)
    switch.Position = UDim2.new(0, 20, 0, yOffset)
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

local function addDropdown(name, callback)
    local dropdown = {}
    local button = addButton(name, function()
        dropdown.Content.Visible = not dropdown.Content.Visible
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

-- ==================== START OF NEW "Kill" TAB ====================

local Killer = gui:AddTab("Kill") -- Create a new tab named "Kill"

local playerWhitelist = {}
local targetPlayerNames = {}
local autoGoodKarma = false
local autoBadKarma = false
local autoKill = false
local killTarget = false
local spying = false
local autoEquipPunch = false
local autoPunchNoAnim = false

local titleLabel = Killer:AddLabel("Select damage or durability pet")
titleLabel.TextSize = 18
titleLabel.Font = Enum.Font.Merriweather
titleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)

local dropdown = Killer:AddDropdown("Select Pet", function(text)
    local petsFolder = game.Players.LocalPlayer.petsFolder
    for _, folder in pairs(petsFolder:GetChildren()) do
        if folder:IsA("Folder") then
            for _, pet in pairs(folder:GetChildren()) do
                game:GetService("ReplicatedStorage").rEvents.equipPetEvent:FireServer("unequipPet", pet)
            end
        end
    end
    task.wait(0.2)
    local petName = text
    local petsToEquip = {}
    for _, pet in pairs(game.Players.LocalPlayer.petsFolder.Unique:GetChildren()) do
        if pet.Name == petName then
            table.insert(petsToEquip, pet)
        end
    end
    local maxPets = 8
    local equippedCount = math.min(#petsToEquip, maxPets)
    for i = 1, equippedCount do
        game:GetService("ReplicatedStorage").rEvents.equipPetEvent:FireServer("equipPet", petsToEquip[i])
        task.wait(0.1)
    end
end)

local Wild_Wizard = dropdown:Add("Wild Wizard")
local Powerful_Monster = dropdown:Add("Mighty Monster")

-- Auto Good Karma switch
Killer:AddSwitch("Auto Good Karma", function(bool)
    autoGoodKarma = bool
    task.spawn(function()
        while autoGoodKarma do
            local playerChar = LocalPlayer.Character
            local rightHand = playerChar and playerChar:FindFirstChild("RightHand")
            local leftHand = playerChar and playerChar:FindFirstChild("LeftHand")
            if playerChar and rightHand and leftHand then
                for _, target in ipairs(Players:GetPlayers()) do
                    if target ~= LocalPlayer then
                        local evilKarma = target:FindFirstChild("evilKarma")
                        local goodKarma = target:FindFirstChild("goodKarma")
                        if evilKarma and goodKarma and evilKarma:IsA("IntValue") and goodKarma:IsA("IntValue") and evilKarma.Value > goodKarma.Value then
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
            task.wait(0.01)
        end
    end)
end)

-- Auto Bad Karma switch
Killer:AddSwitch("Auto Bad Karma", function(bool)
    autoBadKarma = bool
    task.spawn(function()
        while autoBadKarma do
            local playerChar = LocalPlayer.Character
            local rightHand = playerChar and playerChar:FindFirstChild("RightHand")
            local leftHand = playerChar and playerChar:FindFirstChild("LeftHand")
            if playerChar and rightHand and leftHand then
                for _, target in ipairs(Players:GetPlayers()) do
                    if target ~= LocalPlayer then
                        local evilKarma = target:FindFirstChild("evilKarma")
                        local goodKarma = target:FindFirstChild("goodKarma")
                        if evilKarma and goodKarma and evilKarma:IsA("IntValue") and goodKarma:IsA("IntValue") and goodKarma.Value > evilKarma.Value then
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
            task.wait(0.01)
        end
    end)
end)

-- Auto Whitelist Friends toggle
local playerWhitelist = {}
local friendWhitelistActive = false
Killer:AddSwitch("Auto Whitelist Friends", function(state)
    friendWhitelistActive = state
    if state then
        for _, player in ipairs(Players:GetPlayers()) do
            if player ~= LocalPlayer and LocalPlayer:IsFriendsWith(player.UserId) then
                playerWhitelist[player.Name] = true
            end
        end
        Players.PlayerAdded:Connect(function(player)
            if friendWhitelistActive and player ~= LocalPlayer and LocalPlayer:IsFriendsWith(player.UserId) then
                playerWhitelist[player.Name] = true
            end
        end)
    else
        for name in pairs(playerWhitelist) do
            local friend = Players:FindFirstChild(name)
            if friend and LocalPlayer:IsFriendsWith(friend.UserId) then
                playerWhitelist[name] = nil
            end
        end
    end
end)

-- Whitelist and UnWhitelist textboxes
Killer:AddTextBox("Whitelist", function(text)
    local target = Players:FindFirstChild(text)
    if target then
        playerWhitelist[target.Name] = true
    end
end)

Killer:AddTextBox("UnWhitelist", function(text)
    local target = Players:FindFirstChild(text)
    if target then
        playerWhitelist[target.Name] = nil
    end
end)

-- Auto Kill toggle
Killer:AddSwitch("Auto Kill", function(bool)
    autoKill = bool
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
                for _, target in ipairs(Players:GetPlayers()) do
                    if target ~= LocalPlayer and not playerWhitelist[target.Name] then
                        local targetChar = target.Character
                        local rootPart = targetChar and targetChar:FindFirstChild("HumanoidRootPart")
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
end)

-- Target selection dropdown
local targetPlayerNames = {}
local selectedTarget = nil
local targetDropdown = Killer:AddDropdown("Select Target", function(displayName)
    for _, player in ipairs(Players:GetPlayers()) do
        if player.DisplayName == displayName then
            if not table.find(targetPlayerNames, player.Name) then
                table.insert(targetPlayerNames, player.Name)
            end
            selectedTarget = player.Name
        end
    end
end)

-- Remove selected target button
Killer:AddButton("Remove Selected Target", function()
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

-- Populate initial target list
for _, player in ipairs(Players:GetPlayers()) do
    if player ~= LocalPlayer then
        targetDropdown:Add(player.DisplayName)
    end
end
Players.PlayerAdded:Connect(function(player)
    if player ~= LocalPlayer then
        targetDropdown:Add(player.DisplayName)
    end
end)
Players.PlayerRemoving:Connect(function(player)
    -- Refresh list
    targetDropdown:Clear()
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer then
            targetDropdown:Add(plr.DisplayName)
        end
    end
    -- Remove from targetPlayerNames if present
    for i = #targetPlayerNames, 1, -1 do
        if targetPlayerNames[i] == player.Name then
            table.remove(targetPlayerNames, i)
        end
    end
end)

-- Start Kill Target switch
local killTarget = false
Killer:AddSwitch("Start Kill Target", function(state)
    killTarget = state
    task.spawn(function()
        while killTarget do
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
end)

-- View Player / Spy
local spying = false
local targetPlayerName = nil
local spyTargetDropdown = Killer:AddDropdown("Select View Target", function(displayName)
    for _, player in ipairs(Players:GetPlayers()) do
        if player.DisplayName == displayName then
            targetPlayerName = player.Name
            break
        end
    end
end)

for _, player in ipairs(Players:GetPlayers()) do
    if player ~= LocalPlayer then
        spyTargetDropdown:Add(player.DisplayName)
    end
end

Players.PlayerAdded:Connect(function(player)
    if player ~= LocalPlayer then
        spyTargetDropdown:Add(player.DisplayName)
    end
end)

Players.PlayerRemoving:Connect(function(player)
    -- Refresh list
    spyTargetDropdown:Clear()
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer then
            spyTargetDropdown:Add(plr.DisplayName)
        end
    end
end)

Killer:AddSwitch("View Player", function(bool)
    spying = bool
    if not spying then
        local cam = workspace.CurrentCamera
        cam.CameraSubject = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") or LocalPlayer
        return
    end
    task.spawn(function()
        while spying do
            local target = Players:FindFirstChild(targetPlayerName)
            if target and target ~= LocalPlayer then
                local humanoid = target.Character and target.Character:FindFirstChild("Humanoid")
                if humanoid then
                    workspace.CurrentCamera.CameraSubject = humanoid
                end
            end
            task.wait(0.1)
        end
    end)
end)

-- Remove Punch Anim
Killer:AddButton("Remove Punch Anim", function()
    local blockedAnimations = {
        ["rbxassetid://3638729053"] = true,
        ["rbxassetid://3638767427"] = true,
    }
    local function setupAnimationBlocking()
        local char = game.Players.LocalPlayer.Character
        if not char or not char:FindFirstChild("Humanoid") then return end
        local humanoid = char:FindFirstChild("Humanoid")
        for _, track in pairs(humanoid:GetPlayingAnimationTracks()) do
            if track.Animation then
                local animId = track.Animation.AnimationId
                local animName = track.Name:lower()
                if blockedAnimations[animId] or
                   animName:match("punch") or
                   animName:match("attack") or
                   animName:match("right") then
                    track:Stop()
                end
            end
        end
        if not _G.AnimBlockConnection then
            local connection = humanoid.AnimationPlayed:Connect(function(track)
                if track.Animation then
                    local animId = track.Animation.AnimationId
                    local animName = track.Name:lower()
                    if blockedAnimations[animId] or
                       animName:match("punch") or
                       animName:match("attack") or
                       animName:match("right") then
                        track:Stop()
                    end
                end
            end)
            _G.AnimBlockConnection = connection
        end
    end
    setupAnimationBlocking()
end)

-- Auto Equip Punch
Killer:AddSwitch("Auto Equip Punch", function(state)
    autoEquipPunch = state
    task.spawn(function()
        while autoEquipPunch do
            local punch = LocalPlayer.Backpack:FindFirstChild("Punch")
            if punch then
                punch.Parent = LocalPlayer.Character
            end
            task.wait(0.1)
        end
    end)
end)

-- Auto Punch [without animation]
Killer:AddSwitch("Auto Punch [without animation ]", function(state)
    autoPunchNoAnim = state
    task.spawn(function()
        while autoPunchNoAnim do
            local punch = LocalPlayer.Backpack:FindFirstChild("Punch") or LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Punch")
            if punch then
                if punch.Parent ~= LocalPlayer.Character then
                    punch.Parent = LocalPlayer.Character
                end
                LocalPlayer.muscleEvent:FireServer("punch", "rightHand")
                LocalPlayer.muscleEvent:FireServer("punch", "leftHand")
            else
                autoPunchNoAnim = false
            end
            task.wait(0.01)
        end
    end)
end)

-- Auto Punch toggle
Killer:AddSwitch("Auto Punch", function(state)
    _G.fastHitActive = state
    if state then
        task.spawn(function()
            while _G.fastHitActive do
                local punch = LocalPlayer.Backpack:FindFirstChild("Punch")
                if punch then
                    punch.Parent = LocalPlayer.Character
                    if punch:FindFirstChild("attackTime") then
                        punch.attackTime.Value = 0
                    end
                end
                task.wait(0.1)
            end
        end)
        task.spawn(function()
            while _G.fastHitActive do
                local punch = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Punch")
                if punch then
                    punch:Activate()
                end
                task.wait(0.1)
            end
        end)
    else
        local punch = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Punch")
        if punch then
            punch.Parent = LocalPlayer.Backpack
        end
    end
end)

-- God Mode (Good Mode)
local godModeToggle = false
Killer:AddSwitch("Good mode", function(State)
    godModeToggle = State
    if State then
        task.spawn(function()
            while godModeToggle do
                game:GetService("ReplicatedStorage").rEvents.brawlEvent:FireServer("joinBrawl")
                task.wait()
            end
        end)
    end
end)

-- Follow System
local following = false
local followTarget = nil
local function followPlayer(target)
    local myHRP = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
    local targetHRP = target.Character and target.Character:FindFirstChild("HumanoidRootPart")
    if myHRP and targetHRP then
        local followPos = targetHRP.Position - (targetHRP.CFrame.LookVector * 3)
        myHRP.CFrame = CFrame.new(followPos, targetHRP.Position)
    end
end

local function populateFollowList()
    -- Assuming you have a dropdown for follow list
    -- Implement as needed
end

local followDropdown = Killer:AddDropdown("Teleport player", function(displayName)
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr.DisplayName == displayName then
            followTarget = plr
            following = true
            printMsg("Started following " .. plr.Name)
        end
    end
end)

-- populate follow list initially
for _, player in ipairs(Players:GetPlayers()) do
    if player ~= LocalPlayer then
        followDropdown:Add(player.DisplayName)
    end
end

Players.PlayerAdded:Connect(function()
    -- refresh list
    followDropdown:Clear()
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer then
            followDropdown:Add(plr.DisplayName)
        end
    end
end)

Players.PlayerRemoving:Connect(function()
    -- refresh list
    followDropdown:Clear()
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer then
            followDropdown:Add(plr.DisplayName)
        end
    end
    if followTarget and followTarget.Parent then
        if followTarget.Parent == plr then
            followTarget = nil
            following = false
        end
    end
end)

-- Follow loop
task.spawn(function()
    while wait(0.01) do
        if following and followTarget then
            followPlayer(followTarget)
        end
    end
end)

-- Auto Slams
local autoSlams = false
Killer:AddSwitch("auto slams", function(state)
    autoSlams = state
    if state then
        task.spawn(function()
            while autoSlams do
                local player = LocalPlayer
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

-- External scripts button
local urls = {
    "https://raw.githubusercontent.com/SadOz8/Stuffs/refs/heads/main/Crack",
    "https://raw.githubusercontent.com/SadOz8/Stuffs/refs/heads/main/Crack2",
    "https://raw.githubusercontent.com/SadOz8/Stuffs/refs/heads/main/Crack4",
    "https://raw.githubusercontent.com/SadOz8/Stuffs/refs/heads/main/Crack5",
    "https://raw.githubusercontent.com/SadOz8/Stuffs/refs/heads/main/Crack6"
}
Killer:AddButton("Touch Me!", function()
    for _, url in ipairs(urls) do
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
end)

-- Time change dropdown
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
local timeDropdown = Killer:AddDropdown("change time", function(selection)
    changeTime(selection)
end)
for _, option in ipairs(timeOptions) do
    timeDropdown:Add(option)
end
