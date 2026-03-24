-- SERVICES
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local Lighting = game:GetService("Lighting")
local Workspace = game:GetService("Workspace")
local LocalPlayer = Players.LocalPlayer

-- CREATE GUI
local gui = Instance.new("ScreenGui", LocalPlayer:WaitForChild("PlayerGui"))
gui.Name = "MyFeatureUI"

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 650, 0, 500)
frame.Position = UDim2.new(0.5, -325, 0.5, -250)
frame.BackgroundColor3 = Color3.fromRGB(0, 10, 0)
frame.BorderSizePixel = 0

-- MAKE DRAGGABLE FUNCTION
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

-- TITLE
local title = Instance.new("TextLabel", frame)
title.Size = UDim2.new(1, 0, 0, 50)
title.BackgroundColor3 = Color3.fromRGB(0, 20, 0)
title.TextColor3 = Color3.new(1, 1, 1)
title.Text = "Invincible Private Killing"
title.Font = Enum.Font.SourceSansBold
title.TextSize = 20

local yOffset = 60

-- Helper functions
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

-- VARIABLES
local playerWhitelist = {}
local targetPlayerNames = {}
local autoGoodKarma = false
local autoBadKarma = false
local autoKill = false
local killTarget = false
local spying = false
local autoEquipPunch = false
local autoPunchNoAnim = false

local function printMsg(msg)
    print(msg)
end

-- DROPDOWNS
local KillDropdown = addDropdown("Kill", function() end)
local TargetDropdown = addDropdown("Select Target", function(name)
    if not table.find(targetPlayerNames, name) then
        table.insert(targetPlayerNames, name)
    end
end)

-- Add initial players to target dropdown
local function populateTargetDropdown()
    TargetDropdown:Clear()
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer then
            TargetDropdown:Add(player.DisplayName)
        end
    end
end
populateTargetDropdown()

Players.PlayerAdded:Connect(function(player)
    populateTargetDropdown()
end)
Players.PlayerRemoving:Connect(function(player)
    populateTargetDropdown()
    -- Remove from targetPlayerNames if present
    for i, name in ipairs(targetPlayerNames) do
        if name == player.DisplayName then
            table.remove(targetPlayerNames, i)
        end
    end
end)

-- WHITELIST & UNWHITELIST BUTTONS (simple example)
addButton("Whitelist", function()
    -- Example: whitelist a specific player
    -- You can enhance this with input dialog
    local playerName = "PlayerName" -- Replace with actual input
    local player = Players:FindFirstChild(playerName)
    if player then
        playerWhitelist[player.Name] = true
        printMsg("Whitelisted "..player.Name)
    end
end)

addButton("UnWhitelist", function()
    -- Example: unwhitelist specific player
    local playerName = "PlayerName" -- Replace with actual input
    playerWhitelist[playerName] = nil
    printMsg("UnWhitelisted "..playerName)
end)

-- AUTO GOOD KARMA
local autoGoodKarmaSwitch = addSwitch("Auto Good Karma", function(state)
    autoGoodKarma = state
    if state then
        task.spawn(function()
            while autoGoodKarma do
                local char = LocalPlayer.Character
                local rightHand = char and char:FindFirstChild("RightHand")
                local leftHand = char and char:FindFirstChild("LeftHand")
                if rightHand and leftHand then
                    for _, target in ipairs(Players:GetPlayers()) do
                        if target ~= LocalPlayer and target.Character then
                            local evilKarma = target.Character:FindFirstChild("evilKarma")
                            local goodKarma = target.Character:FindFirstChild("goodKarma")
                            if evilKarma and goodKarma and evilKarma:IsA("IntValue") and goodKarma:IsA("IntValue") then
                                if evilKarma.Value > goodKarma.Value then
                                    local rootPart = target.Character:FindFirstChild("HumanoidRootPart")
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

-- AUTO BAD KARMA
local autoBadKarmaSwitch = addSwitch("Auto Bad Karma", function(state)
    autoBadKarma = state
    if state then
        task.spawn(function()
            while autoBadKarma do
                local char = LocalPlayer.Character
                local rightHand = char and char:FindFirstChild("RightHand")
                local leftHand = char and char:FindFirstChild("LeftHand")
                if rightHand and leftHand then
                    for _, target in ipairs(Players:GetPlayers()) do
                        if target ~= LocalPlayer and target.Character then
                            local evilKarma = target.Character:FindFirstChild("evilKarma")
                            local goodKarma = target.Character:FindFirstChild("goodKarma")
                            if evilKarma and goodKarma and evilKarma:IsA("IntValue") and goodKarma:IsA("IntValue") then
                                if goodKarma.Value > evilKarma.Value then
                                    local rootPart = target.Character:FindFirstChild("HumanoidRootPart")
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

-- AUTO WHITELIST FRIENDS
local friendWhitelist = false
local function toggleFriendWhitelist(state)
    friendWhitelist = state
    if state then
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
        for name, _ in pairs(playerWhitelist) do
            playerWhitelist[name] = nil
        end
    end
end
local whitelistSwitch = addSwitch("Auto Whitelist Friends", toggleFriendWhitelist)

-- AUTO KILL
local autoKillSwitch = addSwitch("Auto Kill", function(state)
    autoKill = state
    if state then
        task.spawn(function()
            while autoKill do
                local char = LocalPlayer.Character
                local rightHand = char and char:FindFirstChild("RightHand")
                local leftHand = char and char:FindFirstChild("LeftHand")
                local punch = LocalPlayer.Backpack:FindFirstChild("Punch")
                if punch and not char:FindFirstChild("Punch") then
                    punch.Parent = char
                end
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
                task.wait(0.05)
            end
        end)
    end
end)

-- TARGET SELECTION
local function addTarget(name, callback)
    return addDropdown(name, callback)
end

local targetDropdown = addTarget("Select Target", function(displayName)
    for _, player in ipairs(Players:GetPlayers()) do
        if player.DisplayName == displayName then
            if not table.find(targetPlayerNames, player.Name) then
                table.insert(targetPlayerNames, player.Name)
            end
            return
        end
    end
end)

-- Add button to remove selected target
addButton("Remove Selected Target", function()
    local selected = targetDropdown:GetSelected()
    if selected then
        for i, name in ipairs(targetPlayerNames) do
            if name == selected then
                table.remove(targetPlayerNames, i)
                break
            end
        end
        -- refresh dropdown
        targetDropdown:Clear()
        for _, name in ipairs(targetPlayerNames) do
            -- find display name
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
    end
end)

-- Start Kill Target toggle
local killTarget = false
local killSwitch = addSwitch("Start Kill Target", function(state)
    killTarget = state
    if state then
        task.spawn(function()
            while killTarget do
                local char = LocalPlayer.Character
                local rightHand = char and char:FindFirstChild("RightHand")
                local leftHand = char and char:FindFirstChild("LeftHand")
                local punch = LocalPlayer.Backpack:FindFirstChild("Punch")
                if punch and not char:FindFirstChild("Punch") then
                    punch.Parent = char
                end
                for _, name in ipairs(targetPlayerNames) do
                    local target = Players:FindFirstChild(name)
                    if target and target.Character then
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
    end
end)

-- VIEW PLAYER (SPY)
local spying = false
local function toggleSpy(state)
    spying = state
    if not spying then
        workspace.CurrentCamera.CameraSubject = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") or LocalPlayer
        return
    end
    task.spawn(function()
        while spying do
            -- Assuming you want to follow a specific target
            -- You need to have a variable for that
            -- For demo, just follow the first in targetPlayerNames
            local targetName = targetPlayerNames[1]
            if targetName then
                local target = Players:FindFirstChild(targetName)
                if target and target ~= LocalPlayer and target.Character then
                    local humanoid = target.Character:FindFirstChild("Humanoid")
                    if humanoid then
                        workspace.CurrentCamera.CameraSubject = humanoid
                    end
                end
            end
            task.wait(0.1)
        end
    end)
end
local spySwitch = addSwitch("View Player", toggleSpy)

-- REMOVE PUNCH ANIM
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

-- RECOVER PUNCH (remove blocking)
addButton("Recover Punch Anim", function()
    if _G.AnimBlockConnection then _G.AnimBlockConnection:Disconnect() _G.AnimBlockConnection = nil end
end)

-- Auto Equip Punch
local autoEquipPunch = false
local function toggleAutoEquipPunch(state)
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
end
local autoEquipSwitch = addSwitch("Auto Equip Punch", toggleAutoEquipPunch)

-- Auto Punch without animation
local autoPunchNoAnim = false
local function toggleAutoPunchNoAnim(state)
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
end
local autoPunchNoAnimSwitch = addSwitch("Auto Punch [without animation]", toggleAutoPunchNoAnim)

-- Fast Punch
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

local function toggleFastPunch(state)
    _G.FastPunch = state
    if state then
        autoFastPunch()
    end
end
local fastPunchSwitch = addSwitch("Auto Punch (Fast)", toggleFastPunch)

-- GOD MODE (Good Mode)
local godMode = false
local function toggleGodMode(state)
    godMode = state
    if state then
        task.spawn(function()
            while godMode do
                game:GetService("ReplicatedStorage").rEvents.brawlEvent:FireServer("joinBrawl")
                task.wait(1)
            end
        end)
    end
end
local godModeSwitch = addSwitch("Good mode", toggleGodMode)

-- FOLLOW SYSTEM
local following = false
local followTarget = nil

local function followPlayer(target)
    local myHRP = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
    local targetHRP = target.Character and target.Character:FindFirstChild("HumanoidRootPart")
    if myHRP and targetHRP then
        myHRP.CFrame = CFrame.new(targetHRP.Position - targetHRP.CFrame.LookVector * 3, targetHRP.Position)
    end
end

local function populateFollowList()
    followDropdown:Clear()
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer then
            followDropdown:Add(plr.DisplayName)
        end
    end
end

local followDropdown = addDropdown("Teleport player", function(displayName)
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr.DisplayName == displayName then
            followTarget = plr
            following = true
            printMsg("Started following " .. plr.Name)
        end
    end
end)

-- populate follow list initially
populateFollowList()

-- update follow list on join/leave
Players.PlayerAdded:Connect(function() populateFollowList() end)
Players.PlayerRemoving:Connect(function() populateFollowList() end)

-- Stop following button
addButton("Dejar de Seguir", function()
    following = false
    followTarget = nil
    printMsg("Stopped following")
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

-- When respawn, re-follow if needed
LocalPlayer.CharacterAdded:Connect(function()
    task.wait(1)
    if following and followTarget then
        followPlayer(followTarget)
    end
end)

-- AUTO SLAMS (auto slams)
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

-- TOUCH ME! (execute external scripts)
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
                    warn("[Pegar Muerto] Error executing raw: ", url, err)
                end
            else
                warn("[Pegar Muerto] Failed to load: ", url)
            end
        end)
    end
end)

-- CHANGE TIME (Lighting)
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

-- Time dropdown
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

-- END of script
