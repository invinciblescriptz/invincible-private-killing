-- Services
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local Lighting = game:GetService("Lighting")
local Workspace = game:GetService("Workspace")
local LocalPlayer = Players.LocalPlayer

-- Create main GUI
local gui = Instance.new("ScreenGui", LocalPlayer:WaitForChild("PlayerGui"))
gui.Name = "MyFeatureUI"

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 650, 0, 500)
frame.Position = UDim2.new(0.5, -325, 0.5, -250)
frame.BackgroundColor3 = Color3.fromRGB(0, 10, 0)
frame.BorderSizePixel = 0

-- Make GUI draggable
local function makeDraggable(frame)
    local dragging = false
    local dragInput, dragStartPos, startPos

    local function update(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement then
            local delta = input.Position - dragStartPos
            frame.Position = UDim2.new(
                startPos.X.Scale,
                startPos.X.Offset + delta.X,
                startPos.Y.Scale,
                startPos.Y.Offset + delta.Y
            )
        end
    end

    frame.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            dragStartPos = input.Position
            startPos = frame.Position
            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then
                    dragging = false
                end
            end)
        end
    end)

    UserInputService.InputChanged:Connect(function(input)
        if dragging and input == dragInput then
            update(input)
        end
    end)
end

makeDraggable(frame)

-- Title Label
local title = Instance.new("TextLabel", frame)
title.Size = UDim2.new(1, 0, 0, 50)
title.BackgroundColor3 = Color3.fromRGB(0, 20, 0)
title.TextColor3 = Color3.new(1, 1, 1)
title.Text = "Invincible Private Killing"
title.Font = Enum.Font.SourceSansBold
title.TextSize = 20

local yOffset = 60

-- Helper functions for UI controls
local function addButton(text, callback)
    local btn = Instance.new("TextButton", frame)
    btn.Size = UDim2.new(0, 300, 0, 40)
    btn.Position = UDim2.new(0, 20, 0, yOffset)
    btn.Text = text
    btn.BackgroundColor3 = Color3.fromRGB(0, 50, 0)
    btn.TextColor3 = Color3.new(1, 1, 1)
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
    switch.TextColor3 = Color3.new(1, 1, 1)
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
    local dropdownFrame = Instance.new("Frame", frame)
    dropdownFrame.Size = UDim2.new(0, 300, 0, 40)
    dropdownFrame.Position = UDim2.new(0, 20, 0, yOffset)
    yOffset = yOffset + 45

    local button = addButton(name, function()
        -- Toggle dropdown visibility
        dropdownContent.Visible = not dropdownContent.Visible
    end)

    local dropdownContent = Instance.new("Frame", frame)
    dropdownContent.Size = UDim2.new(0, 300, 0, 0)
    dropdownContent.Position = UDim2.new(0, 20, 0, yOffset)
    dropdownContent.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
    dropdownContent.Visible = false
    local listLayout = Instance.new("UIListLayout", dropdownContent)

    function dropdown:Add(itemName)
        local btn = addButton(itemName, function()
            callback(itemName)
            dropdownContent.Visible = false
        end)
        btn.Parent = dropdownContent
    end

    function dropdown:Clear()
        for _, child in pairs(dropdownContent:GetChildren()) do
            if child:IsA("TextButton") then
                child:Destroy()
            end
        end
    end

    function dropdown:GetSelected()
        return dropdown.SelectedItem
    end

    return dropdown
end

-- Initialize variables
local playerWhitelist = {}
local targetPlayerNames = {}
local autoGoodKarma = false
local autoBadKarma = false
local autoKill = false
local killTarget = false
local spying = false
local autoEquipPunch = false
local autoPunchNoAnim = false
local targetDropdownItems = {}
local availableTargets = {}
local selectedTarget = nil

-- Kill Dropdown
local Killer = addDropdown("Kill", function() end)

-- Add label for pets
local titleLabel = Instance.new("TextLabel", Killer:AddButton("Select damage or durability pet", function() end))
titleLabel.TextSize = 18
titleLabel.Font = Enum.Font.Merriweather
titleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)

-- Helper function to add pets to dropdown
local function addPetDropdown()
    local petsFolder = game.Players.LocalPlayer.petsFolder
    for _, folder in pairs(petsFolder:GetChildren()) do
        if folder:IsA("Folder") then
            for _, pet in pairs(folder:GetChildren()) do
                game:GetService("ReplicatedStorage").rEvents.equipPetEvent:FireServer("unequipPet", pet)
            end
        end
    end
    task.wait(0.2)
    -- Add pets to dropdown
    for _, pet in pairs(game.Players.LocalPlayer.petsFolder.Unique:GetChildren()) do
        Killer:Add(pet.Name)
    end
end
addPetDropdown()

-- Auto Good Karma
local autoGoodKarmaSwitch = addSwitch("Auto Good Karma", function(state)
    autoGoodKarma = state
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
end)

-- Auto Bad Karma
local autoBadKarmaSwitch = addSwitch("Auto Bad Karma", function(state)
    autoBadKarma = state
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
end)

-- Auto Whitelist Friends
local friendWhitelistActive = false
local whitelistSwitch = addSwitch("Auto Whitelist Friends", function(state)
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

-- Whitelist button
addButton("Whitelist", function()
    local targetName = "PlayerName" -- Replace with actual input if needed
    local target = Players:FindFirstChild(targetName)
    if target then
        playerWhitelist[target.Name] = true
    end
end)

-- UnWhitelist button
addButton("UnWhitelist", function()
    local targetName = "PlayerName" -- Replace with actual input if needed
    local target = Players:FindFirstChild(targetName)
    if target then
        playerWhitelist[target.Name] = nil
    end
end)

-- Auto Kill
local autoKillSwitch = addSwitch("Auto Kill", function(state)
    autoKill = state
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

-- Target Dropdown
local targetDropdown = addDropdown("Select Target", function(displayName)
    for _, player in ipairs(Players:GetPlayers()) do
        if player.DisplayName == displayName then
            if not table.find(targetPlayerNames, player.Name) then
                table.insert(targetPlayerNames, player.Name)
            end
            return
        end
    end
end)

-- Button to remove target
addButton("Remove Selected Target", function()
    local selected = targetDropdown:GetSelected()
    if selected then
        for i, name in ipairs(targetPlayerNames) do
            if name == selected then
                table.remove(targetPlayerNames, i)
                break
            end
        end
        targetDropdown:Clear()
        for _, name in ipairs(targetPlayerNames) do
            targetDropdown:Add(name)
        end
    end
end)

-- Populate initial target list
for _, player in ipairs(Players:GetPlayers()) do
    if player ~= LocalPlayer then
        targetDropdown:Add(player.DisplayName)
        targetPlayerNames[#targetPlayerNames + 1] = player.Name
    end
end

-- Update list on join/leave
Players.PlayerAdded:Connect(function(player)
    if player ~= LocalPlayer then
        targetDropdown:Add(player.DisplayName)
        targetPlayerNames[#targetPlayerNames + 1] = player.Name
    end
end)

Players.PlayerRemoving:Connect(function(player)
    -- Remove from list
    for i, name in ipairs(targetPlayerNames) do
        if name == player.Name then
            table.remove(targetPlayerNames, i)
            break
        end
    end
    -- Refresh dropdown
    targetDropdown:Clear()
    for _, name in ipairs(targetPlayerNames) do
        -- Find display name
        local displayName = ""
        for _, plr in ipairs(Players:GetPlayers()) do
            if plr.Name == name then
                displayName = plr.DisplayName
                break
            end
        end
        if displayName ~= "" then
            targetDropdown:Add(displayName)
        end
    end
end)

-- Start Kill Toggle
local killSwitch = addSwitch("Start Kill Target", function(state)
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
            task.wait(0.05)
        end
    end)
end)

-- View Player (Spy)
local spying = false
local spySwitch = addSwitch("View Player", function(state)
    spying = state
    if not spying then
        workspace.CurrentCamera.CameraSubject = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") or LocalPlayer
        return
    end
    task.spawn(function()
        while spying do
            -- Assuming there's a variable targetPlayerName
            -- Replace with your actual target player variable
            -- e.g., local targetPlayerName = "PlayerName"
            local targetPlayerName = nil -- Set this correctly
            if targetPlayerName then
                local target = Players:FindFirstChild(targetPlayerName)
                if target and target ~= LocalPlayer then
                    local humanoid = target.Character and target.Character:FindFirstChild("Humanoid")
                    if humanoid then
                        workspace.CurrentCamera.CameraSubject = humanoid
                    end
                end
            end
            task.wait(0.1)
        end
    end)
end)

-- Remove Punch Anim Button
addButton("Remove Punch Anim", function()
    local blockedAnimations = {
        ["rbxassetid://3638729053"] = true,
        ["rbxassetid://3638767427"] = true,
    }
    local function setupAnimationBlocking()
        local char = LocalPlayer.Character
        if not char or not char:FindFirstChild("Humanoid") then return end
        local humanoid = char:FindFirstChild("Humanoid")
        for _, track in pairs(humanoid:GetPlayingAnimationTracks()) do
            if track.Animation then
                local animId = track.Animation.AnimationId
                local animName = track.Name:lower()
                if blockedAnimations[animId] or animName:match("punch") or animName:match("attack") or animName:match("right") then
                    track:Stop()
                end
            end
        end
        if not _G.AnimBlockConnection then
            _G.AnimBlockConnection = humanoid.AnimationPlayed:Connect(function(track)
                if track.Animation then
                    local animId = track.Animation.AnimationId
                    local animName = track.Name:lower()
                    if blockedAnimations[animId] or animName:match("punch") or animName:match("attack") or animName:match("right") then
                        track:Stop()
                    end
                end
            end)
        end
    end
    setupAnimationBlocking()
end)

-- Recover Punch
addButton("Recover Punch Anim", function()
    if _G.AnimBlockConnection then _G.AnimBlockConnection:Disconnect() _G.AnimBlockConnection = nil end
    if _G.AnimMonitorConnection then _G.AnimMonitorConnection:Disconnect() _G.AnimMonitorConnection = nil end
    if _G.ToolConnections then
        for _, conn in pairs(_G.ToolConnections) do if conn then conn:Disconnect() end end
        _G.ToolConnections = nil
    end
    if _G.BackpackAddedConnection then _G.BackpackAddedConnection:Disconnect() _G.BackpackAddedConnection = nil end
    if _G.CharacterToolAddedConnection then _G.CharacterToolAddedConnection:Disconnect() _G.CharacterToolAddedConnection = nil end
    if _G.CharacterAddedConnection then _G.CharacterAddedConnection:Disconnect() _G.CharacterAddedConnection = nil end
end)

-- Auto Equip Punch
local autoEquipPunch = false
addSwitch("Auto Equip Punch", function(state)
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

-- Auto Punch without animation
local autoPunchNoAnim = false
addSwitch("Auto Punch [without animation ]", function(state)
    autoPunchNoAnim = state
    task.spawn(function()
        while autoPunchNoAnim do
            local punch = LocalPlayer.Backpack:FindFirstChild("Punch") or LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Punch")
            if punch then
                if punch.Parent ~= LocalPlayer.Character then punch.Parent = LocalPlayer.Character end
                -- Fire punch server event
                LocalPlayer.muscleEvent:FireServer("punch", "rightHand")
                LocalPlayer.muscleEvent:FireServer("punch", "leftHand")
            else
                autoPunchNoAnim = false
            end
            task.wait(0.01)
        end
    end)
end)

-- Auto Punch (Fast)
local function autoFastPunch()
    if getgenv().NexusRunning then
        task.spawn(function()
            while _G.FastPunch do
                pcall(function()
                    if muscleEvent then
                        muscleEvent:FireServer("punch", "rightHand")
                        muscleEvent:FireServer("punch", "leftHand")
                    end
                    local char = LocalPlayer.Character
                    if char then
                        local punch = char:FindFirstChild("Punch") or LocalPlayer.Backpack:FindFirstChild("Punch")
                        if punch and punch.Parent ~= char then
                            char.Humanoid:EquipTool(punch)
                        end
                        if punch and punch.Parent == char then
                            local atk = punch:FindFirstChild("attackTime")
                            if atk and atk:IsA("NumberValue") then atk.Value = 0.01 end
                            punch:Activate()
                        end
                    end
                end)
                task.wait(0.085)
            end
        end)
    end
end

-- Fast Punch toggle
local fastPunchSwitch = addSwitch("Fast punch", function(s)
    _G.autoPunchActive = s
    if s then
        autoFastPunch()
    else
        -- stopping logic if needed
    end
end)

-- God Mode (Good Mode)
local godMode = false
local function toggleGodMode(state)
    godMode = state
    if state then
        task.spawn(function()
            while godMode do
                game:GetService("ReplicatedStorage").rEvents.brawlEvent:FireServer("joinBrawl")
                task.wait()
            end
        end)
    end
end
local godModeSwitch = addSwitch("Good mode", toggleGodMode)

-- Follow System
local following = false
local followTarget = nil

local function followPlayer(target)
    local myHRP = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
    local targetHRP = target.Character and target.Character:FindFirstChild("HumanoidRootPart")
    if myHRP and targetHRP then
        local followPos = targetHRP.CFrame * CFrame.new(0, 0, -3)
        myHRP.CFrame = followPos
    end
end

local followDropdown = addDropdown("Teleport player", function(selectedDisplayName)
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr.DisplayName == selectedDisplayName then
            followTarget = plr
            following = true
            print("✅ Started following:", plr.Name)
            break
        end
    end
end)

-- Populate initial list
for _, player in ipairs(Players:GetPlayers()) do
    if player ~= LocalPlayer then
        followDropdown:Add(player.DisplayName)
    end
end

Players.PlayerAdded:Connect(function(player)
    if player ~= LocalPlayer then
        followDropdown:Add(player.DisplayName)
    end
end)

Players.PlayerRemoving:Connect(function(player)
    -- Update list
    followDropdown:Clear()
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer then
            followDropdown:Add(plr.DisplayName)
        end
    end
    if followTarget and followTarget.Name == player.Name then
        followTarget = nil
        following = false
    end
end)

-- Stop following button
addButton("Dejar de Seguir", function()
    following = false
    followTarget = nil
    print("⛔ Stopped following")
end)

-- Follow loop
task.spawn(function()
    while true do
        if following and followTarget and followTarget.Character then
            followPlayer(followTarget)
        end
        task.wait(0.01)
    end
end)

-- When character respawns, re-follow if needed
LocalPlayer.CharacterAdded:Connect(function()
    task.wait(1)
    if following and followTarget then
        followPlayer(followTarget)
    end
end)

-- Auto Slams (auto slams)
local autoSlams = false
local function toggleAutoSlams(state)
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
end
local autoSlamsSwitch = addSwitch("auto slams", toggleAutoSlams)

-- Misc Buttons (e.g., "Combo NaN", "Touch Me!", etc.)
addButton("Combo NaN", function()
    local args = {"changeSize", 0/0}
    game:GetService("ReplicatedStorage"):WaitForChild("rEvents"):WaitForChild("changeSpeedSizeRemote"):InvokeServer(unpack(args))
end)

local urls = {
    "https://raw.githubusercontent.com/SadOz8/Stuffs/refs/heads/main/Crack",
    "https://raw.githubusercontent.com/SadOz8/Stuffs/refs/heads/main/Crack2",
    "https://raw.githubusercontent.com/SadOz8/Stuffs/refs/heads/main/Crack4",
    "https://raw.githubusercontent.com/SadOz8/Stuffs/refs/heads/main/Crack5",
    "https://raw.githubusercontent.com/SadOz8/Stuffs/refs/heads/main/Crack6"
}
addButton("Touch Me!", function()
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

-- Change Time (Lighting)
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

local timeDropdown = addDropdown("change time", function(selection)
    changeTime(selection)
end)
timeDropdown:Add("Morning")
timeDropdown:Add("Noon")
timeDropdown:Add("Afternoon")
timeDropdown:Add("Sunset")
timeDropdown:Add("Night")
timeDropdown:Add("Midnight")
timeDropdown:Add("Dawn")
timeDropdown:Add("Early Morning")
