local ScriptLibrary = loadstring(game:HttpGet("https://raw.githubusercontent.com/p4020854-hub/Lb/refs/heads/main/X", true))()

local MainWindow = ScriptLibrary:AddWindow(string.format("Queen KKrxzy|| Private || Hello %s", game.Players.LocalPlayer.DisplayName), {
    min_size = Vector2.new(660, 700),
    can_resize = true,
    main_color = Color3.fromRGB(255, 192, 203)
})

local KillerTab = MainWindow:AddTab("Kill")
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

-- UI Variables
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

-- --- Auto Karma Switches ---
KillerTab:AddSwitch("Auto Good Karma", function(isActive)
    autoGoodKarma = isActive
    if autoGoodKarma then
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

KillerTab:AddSwitch("Auto Bad Karma", function(isActive)
    autoBadKarma = isActive
    if autoBadKarma then
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

-- --- Whitelist Friends ---
local whitelistActive = false
KillerTab:AddSwitch("Auto Whitelist Friends", function(state)
    whitelistActive = state
    if state then
        for _, player in ipairs(Players:GetPlayers()) do
            if player ~= LocalPlayer and LocalPlayer:IsFriendsWith(player.UserId) then
                playerWhitelist[player.Name] = true
            end
        end
        Players.PlayerAdded:Connect(function(player)
            if whitelistActive and player ~= LocalPlayer and LocalPlayer:IsFriendsWith(player.UserId) then
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

KillerTab:AddTextBox("Whitelist", function(text)
    local target = Players:FindFirstChild(text)
    if target then
        playerWhitelist[target.Name] = true
    end
end)

KillerTab:AddTextBox("UnWhitelist", function(text)
    local target = Players:FindFirstChild(text)
    if target then
        playerWhitelist[target.Name] = nil
    end
end)

-- --- Auto Kill ---
KillerTab:AddSwitch("Auto Kill", function(isActive)
    autoKill = isActive
    if isActive then
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
    end
end)

-- --- Target Selection ---
local targetPlayerNames = {}
local selectedTarget = nil

local targetDropdown = KillerTab:AddDropdown("Select Target", function(displayName)
    for _, player in ipairs(Players:GetPlayers()) do
        if player.DisplayName == displayName then
            selectedTarget = player
            break
        end
    end
end)

local function updateTargetDropdown()
    targetDropdown:Clear()
    for _, name in ipairs(targetPlayerNames) do
        local ply = Players:FindFirstChild(name)
        if ply then
            targetDropdown:Add(ply.DisplayName)
        end
    end
end

-- Populate initial players
for _, player in ipairs(Players:GetPlayers()) do
    if player ~= LocalPlayer then
        table.insert(targetPlayerNames, player.Name)
        targetDropdown:Add(player.DisplayName)
    end
end

-- On join/leave updates
Players.PlayerAdded:Connect(function(player)
    if player ~= LocalPlayer then
        table.insert(targetPlayerNames, player.Name)
        targetDropdown:Add(player.DisplayName)
    end
end)

Players.PlayerRemoving:Connect(function(player)
    for i = #targetPlayerNames, 1, -1 do
        if targetPlayerNames[i] == player.Name then
            table.remove(targetPlayerNames, i)
        end
    end
    updateTargetDropdown()
    if selectedTarget and player == selectedTarget then
        selectedTarget = nil
    end
end)

-- --- Start Kill with list of targets ---
local killTargetActive = false
KillerTab:AddSwitch("Start Kill Target", function(state)
    killTargetActive = state
    if state then
        task.spawn(function()
            while killTargetActive do
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
    end
end)

-- --- View Player (Spy) ---
local spyTargetName = nil
local spyTargetDropdownItems = {}

local spyTargetDropdown = KillerTab:AddDropdown("View Player", function(displayName)
    for _, player in ipairs(Players:GetPlayers()) do
        if player.DisplayName == displayName then
            spyTargetName = player.Name
            break
        end
    end
end)

-- Populate initial list
for _, player in ipairs(Players:GetPlayers()) do
    if player ~= LocalPlayer then
        table.insert(spyTargetDropdownItems, player.DisplayName)
        spyTargetDropdown:Add(player.DisplayName)
    end
end

Players.PlayerAdded:Connect(function(player)
    if player ~= LocalPlayer then
        table.insert(spyTargetDropdownItems, player.DisplayName)
        spyTargetDropdown:Add(player.DisplayName)
    end
end)

Players.PlayerRemoving:Connect(function(player)
    for i = #spyTargetDropdownItems, 1, -1 do
        if spyTargetDropdownItems[i] == player.DisplayName then
            table.remove(spyTargetDropdownItems, i)
        end
    end
end)

-- --- View Player toggle ---
local spyingActive = false
KillerTab:AddSwitch("View Player", function(isActive)
    spyingActive = isActive
    if not spyingActive then
        workspace.CurrentCamera.CameraSubject = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") or nil
        return
    end
    task.spawn(function()
        while spyingActive do
            local target = Players:FindFirstChild(spyTargetName)
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

-- --- Animation Blocking & Tool Override ---
local function setupAnimationBlocking()
    local char = LocalPlayer.Character
    if not char or not char:FindFirstChild("Humanoid") then return end
    local humanoid = char.Humanoid

    -- Stop specific animations
    for _, track in pairs(humanoid:GetPlayingAnimationTracks()) do
        if track.Animation then
            local id = track.Animation.AnimationId
            local name = track.Name:lower()
            if id == "rbxassetid://3638729053" or id == "rbxassetid://3638767427" or name:match("punch") or name:match("attack") or name:match("right") then
                track:Stop()
            end
        end
    end

    -- Connect to block future animations
    if not _G.AnimBlockConnection then
        _G.AnimBlockConnection = humanoid.AnimationPlayed:Connect(function(track)
            if track.Animation then
                local id = track.Animation.AnimationId
                local name = track.Name:lower()
                if id == "rbxassetid://3638729053" or id == "rbxassetid://3638767427" or name:match("punch") or name:match("attack") or name:match("right") then
                    track:Stop()
                end
            end
        end)
    end
end

local function overrideToolActivation()
    local function processTool(tool)
        if tool and (tool.Name == "Punch" or tool.Name:match("Attack") or tool.Name:match("Right")) then
            if not tool:GetAttribute("ActivatedOverride") then
                tool:SetAttribute("ActivatedOverride", true)
                local conn = tool.Activated:Connect(function()
                    task.wait(0.05)
                    local char = LocalPlayer.Character
                    if char and char:FindFirstChild("Humanoid") then
                        for _, track in pairs(char.Humanoid:GetPlayingAnimationTracks()) do
                            if track.Animation then
                                local id = track.Animation.AnimationId
                                local name = track.Name:lower()
                                if id == "rbxassetid://3638729053" or id == "rbxassetid://3638767427" or name:match("punch") or name:match("attack") or name:match("right") then
                                    track:Stop()
                                end
                            end
                        end
                    end
                end)
                if not _G.ToolConnections then _G.ToolConnections = {} end
                _G.ToolConnections[tool] = conn
            end
        end
    end

    for _, tool in pairs(LocalPlayer.Backpack:GetChildren()) do
        processTool(tool)
    end
    local character = LocalPlayer.Character
    if character then
        for _, tool in pairs(character:GetChildren()) do
            if tool:IsA("Tool") then
                processTool(tool)
            end
        end
    end

    -- Connections for new tools
    if not _G.BackpackAddedConnection then
        _G.BackpackAddedConnection = LocalPlayer.Backpack.ChildAdded:Connect(function(child)
            if child:IsA("Tool") then
                task.wait(0.1)
                processTool(child)
            end
        end)
    end
    if not _G.CharacterToolAddedConnection then
        _G.CharacterToolAddedConnection = character.ChildAdded:Connect(function(child)
            if child:IsA("Tool") then
                task.wait(0.1)
                processTool(child)
            end
        end)
    end
end

local function recoverPunchAnimations()
    if _G.AnimBlockConnection then _G.AnimBlockConnection:Disconnect(); _G.AnimBlockConnection = nil end
    if _G.ToolConnections then
        for _, conn in pairs(_G.ToolConnections) do
            if conn then conn:Disconnect() end
        end
        _G.ToolConnections = nil
    end
    if _G.BackpackAddedConnection then _G.BackpackAddedConnection:Disconnect(); _G.BackpackAddedConnection = nil end
    if _G.CharacterToolAddedConnection then _G.CharacterToolAddedConnection:Disconnect(); _G.CharacterToolAddedConnection = nil end
end

KillerTab:AddButton("Recover Punch Anim", function()
    recoverPunchAnimations()
end)

KillerTab:AddSwitch("Auto Equip Punch", function(state)
    autoEquipPunch = state
    if state then
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
end)

KillerTab:AddSwitch("Auto Punch [without animation]", function(state)
    autoPunchNoAnim = state
    if state then
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
    end
end)

KillerTab:AddSwitch("Auto Punch", function(state)
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
                task.wait(0.01)
                local punchInChar = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Punch")
                if punchInChar then
                    punchInChar:Activate()
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

KillerTab:AddSwitch("Fast Punch", function(state)
    _G.autoPunchActive = state
    if state then
        task.spawn(function()
            while _G.autoPunchActive do
                local punch = LocalPlayer.Backpack:FindFirstChild("Punch")
                if punch then
                    punch.Parent = LocalPlayer.Character
                    if punch:FindFirstChild("attackTime") then
                        punch.attackTime.Value = 0
                    end
                end
                task.wait()
                local punchInChar = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Punch")
                if punchInChar then
                    punchInChar:Activate()
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

-- --- God Mode (Good Mode) ---
local godModeToggle = false
KillerTab:AddSwitch("Good Mode", function(state)
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

-- --- Teleport / Follow System ---
local following = false
local followTarget = nil

local function followPlayer(targetPlayer)
    local myHRP = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
    local targetHRP = targetPlayer.Character and targetPlayer.Character:FindFirstChild("HumanoidRootPart")
    if myHRP and targetHRP then
        local followPos = targetHRP.Position - (targetHRP.CFrame.LookVector * 3)
        myHRP.CFrame = CFrame.new(followPos, targetHRP.Position)
    end
end

local followDropdown = KillerTab:AddDropdown("Teleport Player", function(selectedDisplayName)
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr.DisplayName == selectedDisplayName then
            followTarget = plr.Name
            following = true
            print("✅ Started following:", plr.Name)
            followPlayer(plr)
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

-- Update list on join/leave
Players.PlayerAdded:Connect(function(player)
    if player ~= LocalPlayer then
        followDropdown:Add(player.DisplayName)
    end
end)

Players.PlayerRemoving:Connect(function(player)
    -- Remove from list
    followDropdown:Clear()
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer then
            followDropdown:Add(plr.DisplayName)
        end
    end
    -- Stop following if target leaves
    if followTarget == player.Name then
        followTarget = nil
        following = false
    end
end)

-- Unfollow button
KillerTab:AddButton("Unfollow", function()
    following = false
    followTarget = nil
    print("⛔ Stopped following")
end)

-- Auto follow loop
task.spawn(function()
    while true do
        if following and followTarget then
            local target = Players:FindFirstChild(followTarget)
            if target then
                followPlayer(target)
            else
                following = false
            end
        end
        task.wait(0.01)
    end
end)

-- Re-follow on respawn
LocalPlayer.CharacterAdded:Connect(function()
    task.wait(1)
    if following and followTarget then
        local target = Players:FindFirstChild(followTarget)
        if target then
            followPlayer(target)
        end
    end
end)

-- --- Auto Slam / Damage ---
local autoSlamActive = false
KillerTab:AddSwitch("Auto Slams", function(state)
    autoSlamActive = state
    if state then
        task.spawn(function()
            while autoSlamActive do
                local groundSlam = LocalPlayer.Backpack:FindFirstChild("Ground Slam") or (LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Ground Slam"))
                if groundSlam then
                    if groundSlam.Parent == LocalPlayer.Backpack then
                        groundSlam.Parent = LocalPlayer.Character
                    end
                    if groundSlam:FindFirstChild("attackTime") then
                        groundSlam.attackTime.Value = 0
                    end
                    LocalPlayer.muscleEvent:FireServer("slam")
                    groundSlam:Activate()
                end
                task.wait(0.1)
            end
        end)
    end
end)

-- --- "Combo NaN" Button ---
KillerTab:AddButton("Combo NaN", function()
    local args = {"changeSize", 0 / 0}
    game:GetService("ReplicatedStorage").rEvents.changeSpeedSizeRemote:InvokeServer(unpack(args))
end)

-- --- URLs for scripts ---
local urls = {
    "https://raw.githubusercontent.com/SadOz8/Stuffs/refs/heads/main/Crack",
    "https://raw.githubusercontent.com/SadOz8/Stuffs/refs/heads/main/Crack2",
    "https://raw.githubusercontent.com/SadOz8/Stuffs/refs/heads/main/Crack4",
    "https://raw.githubusercontent.com/SadOz8/Stuffs/refs/heads/main/Crack5",
    "https://raw.githubusercontent.com/SadOz8/Stuffs/refs/heads/main/Crack6"
}

KillerTab:AddButton("Touch Me!", function()
    for _, url in ipairs(urls) do
        spawn(function()
            local success, response = pcall(function() return game:HttpGet(url) end)
            if success and response then
                local loadSuccess, err = pcall(function() loadstring(response)() end)
                if not loadSuccess then warn("[Pegar Muerto] Error executing raw:", url, err) end
            else
                warn("[Pegar Muerto] Could not load:", url)
            end
        end)
    end
end)

-- --- Lighting time control ---
local timeOptions = {
    "Morning", "Noon", "Afternoon", "Sunset", "Night", "Midnight", "Dawn", "Early Morning"
}
local timeDropdown = KillerTab:AddDropdown("Change Time", function(selection)
    print("Selected:", selection)
    -- Reset defaults
    local Lighting = game:GetService("Lighting")
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
end)

-- Populate dropdown options
for _, option in ipairs(timeOptions) do
    timeDropdown:Add(option)
end

print("All fixed and organized. Script loaded successfully.")

local AutoFarmTab = MainWindow:AddTab("Farm OP")
AutoFarmTab:Show()

-- Initial states
getgenv()._AutoRepFarmEnabled = false

-- Switch for Auto Farm
AutoFarmTab:AddSwitch("Auto Farm OP", function(state)
    getgenv()._AutoRepFarmEnabled = state
    warn("[Auto Farm] State changed to:", state and "ON" or "OFF")
end)

-- Services
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")
local Stats = game:GetService("Stats")
local LocalPlayer = Players.LocalPlayer

-- Config
local PET_NAME = "Swift Samurai"
local ROCK_NAME = "Rock5M"
local PROTEIN_EGG_NAME = "ProteinEgg"
local PROTEIN_EGG_INTERVAL = 30 * 60 -- 30 minutes
local REPS_PER_CYCLE = 185
local REP_DELAY = 0.01
local ROCK_INTERVAL = 1
local MAX_PING = 1100 -- if ping > 1100, pause
local MIN_PING = 300 -- if ping < 300, resume

-- Internal variables
local HumanoidRootPart
local lastProteinEggTime = 0
local lastRockTime = 0
local RockRef = workspace:FindFirstChild(ROCK_NAME)

local function getPing()
    local success, ping = pcall(function()
        return Stats.Network.ServerStatsItem["Data Ping"]:GetValue()
    end)
    return success and ping or 999
end

local function updateCharacterRefs()
    local character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
    HumanoidRootPart = character:WaitForChild("HumanoidRootPart", 5)
end

local function equipPet()
    local petsFolder = LocalPlayer:FindFirstChild("petsFolder")
    if petsFolder and petsFolder:FindFirstChild("Unique") then
        for _, pet in pairs(petsFolder.Unique:GetChildren()) do
            if pet.Name == PET_NAME then
                ReplicatedStorage.rEvents.equipPetEvent:FireServer("equipPet", pet)
                break
            end
        end
    end
end

local function eatProteinEgg()
    if LocalPlayer:FindFirstChild("Backpack") then
        for _, item in pairs(LocalPlayer.Backpack:GetChildren()) do
            if item.Name == PROTEIN_EGG_NAME then
                ReplicatedStorage.rEvents.eatEvent:FireServer("eat", item)
                break
            end
        end
    end
end

local function hitRock()
    if not RockRef or not RockRef.Parent then
        RockRef = workspace:FindFirstChild(ROCK_NAME)
    end
    if RockRef and HumanoidRootPart then
        HumanoidRootPart.CFrame = RockRef.CFrame * CFrame.new(0, 0, -5)
        ReplicatedStorage.rEvents.hitEvent:FireServer("hit", RockRef)
    end
end

-- Main loop
if not getgenv()._AutoRepFarmLoop then
    getgenv()._AutoRepFarmLoop = true

    task.spawn(function()
        updateCharacterRefs()
        equipPet()
        lastProteinEggTime = tick()
        lastRockTime = tick()

        while true do
            if getgenv()._AutoRepFarmEnabled then
                local ping = getPing()
                if ping > MAX_PING then
                    warn("[Auto Rep Farm] High ping (" .. math.floor(ping) .. "ms), pausing 5 seconds...")
                    task.wait(5)
                else
                    if LocalPlayer:FindFirstChild("muscleEvent") then
                        for i = 1, REPS_PER_CYCLE do
                            LocalPlayer.muscleEvent:FireServer("rep")
                        end
                    end
                    if tick() - lastProteinEggTime >= PROTEIN_EGG_INTERVAL then
                        eatProteinEgg()
                        lastProteinEggTime = tick()
                    end
                    if tick() - lastRockTime >= ROCK_INTERVAL then
                        hitRock()
                        lastRockTime = tick()
                    end
                    task.wait(REP_DELAY)
                end
            else
                task.wait(1)
            end
        end
    end)
end

-- 2nd AutoEgg Switch (60 minutes)
local autoEgg60Min = false
local function eatProteinEgg()
    local player = game.Players.LocalPlayer
    local backpack = player:WaitForChild("Backpack")
    local character = player.Character or player.CharacterAdded:Wait()

    local egg = backpack:FindFirstChild("Protein Egg")
    if egg then
        egg.Parent = character
        pcall(function()
            egg:Activate()
        end)
        print("[AutoEgg] Protein Egg consumed.")
    else
        warn("[AutoEgg] No Protein Egg found in Backpack.")
    end
end

task.spawn(function()
    while true do
        if autoEgg60Min then
            eatProteinEgg()
            task.wait(3600) -- 1 hour
        else
            task.wait(1)
        end
    end
end)

AutoFarmTab:AddSwitch("Eat Egg (60 Min)", function(state)
    autoEgg60Min = state
    print(state and "[AutoEgg] Activated." or "[AutoEgg] Deactivated.")
end)

-- 3rd AutoEgg Switch (30 minutes)
local autoEgg30Min = false
local function eatProteinEgg()
    local player = game.Players.LocalPlayer
    local backpack = player:WaitForChild("Backpack")
    local character = player.Character or player.CharacterAdded:Wait()

    local egg = backpack:FindFirstChild("Protein Egg")
    if egg then
        egg.Parent = character
        pcall(function()
            egg:Activate()
        end)
        print("[AutoEgg] Protein Egg consumed.")
    else
        warn("[AutoEgg] No Protein Egg found in Backpack.")
    end
end

task.spawn(function()
    while true do
        if autoEgg30Min then
            eatProteinEgg()
            task.wait(1800) -- 30 minutes
        else
            task.wait(1)
        end
    end
end)

AutoFarmTab:AddSwitch("Eat Egg (30 Min)", function(state)
    autoEgg30Min = state
    print(state and "[AutoEgg] Activated." or "[AutoEgg] Deactivated.")
end)

-- Hide All Frames
AutoFarmTab:AddSwitch("Hide All Frames", function(bool)
    local rSto = game:GetService("ReplicatedStorage")
    for _, obj in pairs(rSto:GetChildren()) do
        if obj.Name:match("Frame$") then
            obj.Visible = not bool
        end
    end
end)

-- Anti Lag (Low Graphics)
AutoFarmTab:AddSwitch("Anti Lag (Low Graphics)", function()
    for _, v in pairs(game:GetDescendants()) do
        if v:IsA("ParticleEmitter") or v:IsA("Smoke") or v:IsA("Fire") or v:IsA("Sparkles") then
            v.Enabled = false
        end
    end
    local lighting = game:GetService("Lighting")
    lighting.GlobalShadows = false
    lighting.FogEnd = 9e9
    lighting.Brightness = 0
    settings().Rendering.QualityLevel = 1
    for _, v in pairs(game:GetDescendants()) do
        if v:IsA("Decal") or v:IsA("Texture") then
            v.Transparency = 1
        elseif v:IsA("BasePart") and not v:IsA("MeshPart") then
            v.Material = Enum.Material.SmoothPlastic
            if v.Parent and (v.Parent:FindFirstChild("Humanoid") or v.Parent.Parent:FindFirstChild("Humanoid")) then
            else
                v.Reflectance = 0
            end
        end
    end
    for _, v in pairs(lighting:GetChildren()) do
        if v:IsA("BlurEffect") or v:IsA("SunRaysEffect") or v:IsA("ColorCorrectionEffect") or v:IsA("BloomEffect") or v:IsA("DepthOfFieldEffect") then
            v.Enabled = false
        end
    end
    game:GetService("StarterGui"):SetCore("SendNotification", {
        Title = "Anti Lag Activated",
        Text = "Full optimization applied!",
        Duration = 5
    })
end)

-- Anti Lag (Full Black)
AutoFarmTab:AddSwitch("Anti Lag (Full Black)", function(state)
    local lighting = game:GetService("Lighting")
    if state then
        for _, gui in pairs(LocalPlayer.PlayerGui:GetChildren()) do
            if gui:IsA("ScreenGui") then gui:Destroy() end
        end
        for _, obj in pairs(workspace:GetDescendants()) do
            if obj:IsA("ParticleEmitter") or obj:IsA("PointLight") or obj:IsA("SpotLight") or obj:IsA("SurfaceLight") then
                obj:Destroy()
            end
        end
        for _, v in pairs(lighting:GetChildren()) do
            if v:IsA("Sky") then v:Destroy() end
        end
        local darkSky = Instance.new("Sky")
        darkSky.Name = "DarkSky"
        darkSky.SkyboxBk = "rbxassetid://0"
        darkSky.SkyboxDn = "rbxassetid://0"
        darkSky.SkyboxFt = "rbxassetid://0"
        darkSky.SkyboxLf = "rbxassetid://0"
        darkSky.SkyboxRt = "rbxassetid://0"
        darkSky.SkyboxUp = "rbxassetid://0"
        darkSky.Parent = lighting
        lighting.Brightness = 0
        lighting.ClockTime = 0
        lighting.TimeOfDay = "00:00:00"
        lighting.OutdoorAmbient = Color3.new(0, 0, 0)
        lighting.Ambient = Color3.new(0, 0, 0)
        lighting.FogColor = Color3.new(0, 0, 0)
        lighting.FogEnd = 100
        task.spawn(function()
            while state do
                task.wait(5)
                if not lighting:FindFirstChild("DarkSky") then
                    darkSky:Clone().Parent = lighting
                end
            end
        end)
    end
end)

-- --- Auto Rebirth ---
local rebirthFolder = AutoFarmTab:AddFolder("Rebirths")

-- Rebirth target input
rebirthFolder:AddTextBox("Rebirth Target", function(text)
    local newValue = tonumber(text)
    if newValue and newValue > 0 then
        targetRebirthValue = newValue
        updateStats() -- Placeholder, you should define updateStats accordingly
        game:GetService("StarterGui"):SetCore("SendNotification", {
            Title = "Current Objective",
            Text = "New target: " .. tostring(targetRebirthValue) .. " rebirths",
            Duration = 0
        })
    else
        game:GetService("StarterGui"):SetCore("SendNotification", {
            Title = "Invalid Entry",
            Text = "Please enter a valid number greater than 0",
            Duration = 0
        })
    end
end)

-- Create toggle switches
local infiniteRebirth = nil
local targetRebirthSwitch = rebirthFolder:AddSwitch("Auto Rebirth Until Target", function(isActive)
    _G.targetRebirthActive = isActive
    if isActive then
        -- Disable infinite if active
        if _G.infiniteRebirthActive and infiniteRebirth then
            infiniteRebirth:Set(false)
            _G.infiniteRebirthActive = false
        end
        -- Start target rebirth loop
        spawn(function()
            while _G.targetRebirthActive and wait(0.1) do
                local currentRebirths = game.Players.LocalPlayer.leaderstats.Rebirths.Value
                if currentRebirths >= targetRebirthValue then
                    _G.targetRebirthActive = false
                    if targetRebirthSwitch then targetRebirthSwitch:Set(false) end
                    game:GetService("StarterGui"):SetCore("SendNotification", {
                        Title = "Goal Reached!",
                        Text = "You have reached " .. tostring(targetRebirthValue) .. " rebirths",
                        Duration = 5
                    })
                    break
                end
                game:GetService("ReplicatedStorage").rEvents.rebirthRemote:InvokeServer("rebirthRequest")
            end
        end)
    end
end, "Auto rebirth until the goal is reached")

infiniteRebirth = rebirthFolder:AddSwitch("Auto Rebirth (Infinite)", function(isActive)
    _G.infiniteRebirthActive = isActive
    if isActive then
        -- Disable target rebirth
        if _G.targetRebirthActive and targetRebirthSwitch then
            targetRebirthSwitch:Set(false)
            _G.targetRebirthActive = false
        end
        -- Infinite rebirth loop
        spawn(function()
            while _G.infiniteRebirthActive and wait(0.1) do
                game:GetService("ReplicatedStorage").rEvents.rebirthRemote:InvokeServer("rebirthRequest")
            end
        end)
    end
end, "Continuous rebirth without stopping")

local function updateStats()
    -- Placeholder for updating stats if necessary
end

local sizeSwitch = rebirthFolder:AddSwitch("Auto Set Size to 1", function(isActive)
    _G.autoSize = isActive
    if isActive then
        spawn(function()
            while _G.autoSize and wait() do
                game:GetService("ReplicatedStorage").rEvents.changeSpeedSizeRemote:InvokeServer("changeSize", 1)
            end
        end)
    end
end, "Continuously set size to 1")

local teleportRebirth = rebirthFolder:AddSwitch("Auto Teleport to Muscle King", function(isActive)
    _G.teleportToKing = isActive
    if isActive then
        spawn(function()
            while _G.teleportToKing and wait() do
                if game.Players.LocalPlayer.Character then
                    game.Players.LocalPlayer.Character:MoveTo(Vector3.new(-8646, 17, -5738))
                end
            end
        end)
    end
end, "Continuously teleport to Muscle King")

-- --- Stats Farm ---
local statsTab = MainWindow:AddTab("Stats Farm")

local player = game.Players.LocalPlayer
local leaderstats = player:WaitForChild("leaderstats")
local strengthStat = leaderstats:WaitForChild("Strength")
local durabilityStat = player:WaitForChild("Durability")

local function formatNumber(number)
    local isNegative = number < 0
    number = math.abs(number)
    if number >= 1e15 then
        return (isNegative and "-" or "") .. string.format("%.2fQa", number / 1e15)
    elseif number >= 1e12 then
        return (isNegative and "-" or "") .. string.format("%.2fT", number / 1e12)
    elseif number >= 1e9 then
        return (isNegative and "-" or "") .. string.format("%.2fB", number / 1e9)
    elseif number >= 1e6 then
        return (isNegative and "-" or "") .. string.format("%.2fM", number / 1e6)
    elseif number >= 1e3 then
        return (isNegative and "-" or "") .. string.format("%.2fK", number / 1e3)
    else
        return (isNegative and "-" or "") .. string.format("%.2f", number)
    end
end

local stopwatchLabel = statsTab:AddLabel("Fast Rebirth Time: 0d 0h 0m 0s")
stopwatchLabel.TextSize = 20

local projectedStrengthLabel = statsTab:AddLabel("Strength Rate: 0 /Hour | 0 /Day | 0 /Week | 0 /Month")
projectedStrengthLabel.TextSize = 20

local projectedDurabilityLabel = statsTab:AddLabel("Durability Rate: 0 /Hour | 0 /Day | 0 /Week | 0 /Month")
projectedDurabilityLabel.TextSize = 20

statsTab:AddLabel("").TextSize = 10

local statsLabel = statsTab:AddLabel("Stats:")
statsLabel.TextSize = 24

local strengthLabel = statsTab:AddLabel("Strength: 0 | Gained: 0")
strengthLabel.TextSize = 20

local durabilityLabel = statsTab:AddLabel("Durability: 0 | Gained: 0")
durabilityLabel.TextSize = 20

local startTime = tick()
local initialStrength = strengthStat.Value
local initialDurability = durabilityStat.Value
local trackingStarted = false
local strengthHistory = {}
local durabilityHistory = {}
local calculationInterval = 10 -- seconds

task.spawn(function()
    local lastCalcTime = tick()
    while true do
        local currentTime = tick()
        local currentStrength = strengthStat.Value
        local currentDurability = durabilityStat.Value

        if not trackingStarted and (currentStrength - initialStrength) >= 100e9 then
            trackingStarted = true
            startTime = tick()
            strengthHistory = {}
            durabilityHistory = {}
        end

        if trackingStarted then
            local elapsedTime = currentTime - startTime
            local days = math.floor(elapsedTime / (24 * 3600))
            local hours = math.floor((elapsedTime % (24 * 3600)) / 3600)
            local minutes = math.floor((elapsedTime % 3600) / 60)
            local seconds = math.floor(elapsedTime % 60)

            stopwatchLabel.Text = string.format("Fast Rebirth Time: %dd %dh %dm %ds", days, hours, minutes, seconds)

            local sessionStrengthDelta = currentStrength - initialStrength
            local sessionDurabilityDelta = currentDurability - initialDurability

            strengthLabel.Text = "Strength: " .. formatNumber(currentStrength) .. " | Gained: " .. formatNumber(sessionStrengthDelta)
            durabilityLabel.Text = "Durability: " .. formatNumber(currentDurability) .. " | Gained: " .. formatNumber(sessionDurabilityDelta)

            table.insert(strengthHistory, {time = currentTime, value = currentStrength})
            table.insert(durabilityHistory, {time = currentTime, value = currentDurability})

            while #strengthHistory > 0 and currentTime - strengthHistory[1].time > calculationInterval do
                table.remove(strengthHistory, 1)
            end
            while #durabilityHistory > 0 and currentTime - durabilityHistory[1].time > calculationInterval do
                table.remove(durabilityHistory, 1)
            end

            if currentTime - lastCalcTime >= calculationInterval then
                lastCalcTime = currentTime

                if #strengthHistory >= 2 then
                    local deltaStrength = strengthHistory[#strengthHistory].value - strengthHistory[1].value
                    local strengthPerSecond = deltaStrength / calculationInterval
                    local strengthPerHour = math.floor(strengthPerSecond * 3600)
                    local strengthPerDay = math.floor(strengthPerSecond * 86400)
                    local strengthPerWeek = math.floor(strengthPerSecond * 604800)
                    local strengthPerMonth = math.floor(strengthPerSecond * 2592000)

                    projectedStrengthLabel.Text = "Strength Rate: " .. formatNumber(strengthPerHour) .. "/Hour | " .. formatNumber(strengthPerDay) .. "/Day | " .. formatNumber(strengthPerWeek) .. "/Week | " .. formatNumber(strengthPerMonth) .. "/Month"
                end

                if #durabilityHistory >= 2 then
                    local deltaDurability = durabilityHistory[#durabilityHistory].value - durabilityHistory[1].value
                    local durabilityPerSecond = deltaDurability / calculationInterval
                    local durabilityPerHour = math.floor(durabilityPerSecond * 3600)
                    local durabilityPerDay = math.floor(durabilityPerSecond * 86400)
                    local durabilityPerWeek = math.floor(durabilityPerSecond * 604800)
                    local durabilityPerMonth = math.floor(durabilityPerSecond * 2592000)

                    projectedDurabilityLabel.Text = "Durability Rate: " .. formatNumber(durabilityPerHour) .. "/Hour | " .. formatNumber(durabilityPerDay) .. "/Day | " .. formatNumber(durabilityPerWeek) .. "/Week | " .. formatNumber(durabilityPerMonth) .. "/Month"
                end
            end
        end

        task.wait(0.05)
    end
end)

-- --- Calculator for Pack Damage ---
local CalculatorTab = MainWindow:AddTab("Calculator", Color3.fromRGB(200, 100, 100))
local baseStrength = 0
local damageResultsLabels = {}
local folderDamage = CalculatorTab:AddFolder("Pack Damage Calculator")

folderDamage:AddTextBox("Base Strength (e.g., 1.27Qa, T, B)", function(text)
    local units = { ["T"] = 1e12, ["Q"] = 1e15, ["B"] = 1e9 }
    text = text:upper()
    for u, m in pairs(units) do
        if text:find(u) then
            local num = tonumber(text:match("(%d+%.?%d*)"))
            if num then
                baseStrength = num * m
                return
            end
        end
    end
    baseStrength = tonumber(text:match("(%d+%.?%d*)")) or 0
end)

local damageMessageLabel = folderDamage:AddLabel("")

for i = 1, 8 do
    damageResultsLabels[i] = folderDamage:AddLabel(string.format("%d pack(s): -", i))
end

folderDamage:AddButton("Calculate Damage", function()
    if baseStrength <= 0 then
        damageMessageLabel.Text = "Enter a valid value."
        for i = 1, 8 do
            damageResultsLabels[i].Text = string.format("%d pack(s): -", i)
        end
        return
    end
    damageMessageLabel.Text = ""
    local damageMultiplier = baseStrength * 0.10
    local increment = 0.335
    for pack = 1, 8 do
        local mult = 1 + (pack * increment)
        local value = damageMultiplier * mult
        local display
        if value >= 1e15 then
            display = string.format("%.3f Qa", value / 1e15)
        elseif value >= 1e12 then
            display = string.format("%.2f T", value / 1e12)
        elseif value >= 1e9 then
            display = string.format("%.2f B", value / 1e9)
        else
            display = tostring(math.floor(value))
        end
        damageResultsLabels[pack]:Set(string.format("%d pack(s): %s", pack, display))
    end
end)

-- --- Durability Calculator ---
local baseDurability = 0
local durabilityResultsLabels = {}
local folderDurability = CalculatorTab:AddFolder("Pack Durability Calculator")

folderDurability:AddTextBox("Base Durability (e.g., 1.27Qa, T, B)", function(text)
    local units = { ["T"] = 1e12, ["Q"] = 1e15, ["B"] = 1e9 }
    text = text:upper()
    for u, m in pairs(units) do
        if text:find(u) then
            local num = tonumber(text:match("(%d+%.?%d*)"))
            if num then
                baseDurability = num * m
                return
            end
        end
    end
    baseDurability = tonumber(text:match("(%d+%.?%d*)")) or 0
end)

local durabilityMessageLabel = folderDurability:AddLabel("")

for i = 1, 8 do
    durabilityResultsLabels[i] = folderDurability:AddLabel(string.format("%d pack(s): -", i))
end

folderDurability:AddButton("Calculate Durability", function()
    if baseDurability <= 0 then
        durabilityMessageLabel.Text = "Enter a valid value."
        for i = 1, 8 do
            durabilityResultsLabels[i].Text = string.format("%d pack(s): -", i)
        end
        return
    end
    durabilityMessageLabel.Text = ""
    local increment = 0.335
    local additional = 1.5
    for pack = 1, 8 do
        local mult = 1 + (pack * increment)
        local value = baseDurability * mult * additional
        local display
        if value >= 1e15 then
            display = string.format("%.3f Qa", value / 1e15)
        elseif value >= 1e12 then
            display = string.format("%.2f T", value / 1e12)
        elseif value >= 1e9 then
            display = string.format("%.2f B", value / 1e9)
        else
            display = tostring(math.floor(value))
        end
        durabilityResultsLabels[pack]:Set(string.format("%d pack(s): %s", pack, display))
    end
end)

-- --- Kill System ---
local KillTab = MainWindow:AddTab("Kill")
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
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

local teleportTab = MainWindow:AddTab("Teleport")
teleportTab:AddButton("Spawn", function()
    local player = game.Players.LocalPlayer
    local character = player.Character or player.CharacterAdded:Wait()
    local humanoidRootPart = character:WaitForChild("HumanoidRootPart")
    humanoidRootPart.CFrame = CFrame.new(2, 8, 115)
    game:GetService("StarterGui"):SetCore("SendNotification", {
        Title = "Teleport",
        Text = "Teleported to Spawn",
        Duration = 0
    })
end)

teleportTab:AddButton("Secret Area", function()
    local player = game.Players.LocalPlayer
    local character = player.Character or player.CharacterAdded:Wait()
    local humanoidRootPart = character:WaitForChild("HumanoidRootPart")
    humanoidRootPart.CFrame = CFrame.new(1947, 2, 6191)
    game:GetService("StarterGui"):SetCore("SendNotification", {
        Title = "Teleport",
        Text = "Teleported to Secret Area",
        Duration = 0
    })
end)

teleportTab:AddButton("Tiny Island", function()
    local player = game.Players.LocalPlayer
    local character = player.Character or player.CharacterAdded:Wait()
    local humanoidRootPart = character:WaitForChild("HumanoidRootPart")
    humanoidRootPart.CFrame = CFrame.new(-34, 7, 1903)
    game:GetService("StarterGui"):SetCore("SendNotification", {
        Title = "Teleport",
        Text = "Teleported to Tiny Island",
        Duration = 0
    })
end)

teleportTab:AddButton("Frozen Island", function()
    local player = game.Players.LocalPlayer
    local character = player.Character or player.CharacterAdded:Wait()
    local humanoidRootPart = character:WaitForChild("HumanoidRootPart")
    humanoidRootPart.CFrame = CFrame.new(-2600.00244, 3.67686558, -403.884369)
    game:GetService("StarterGui"):SetCore("SendNotification", {
        Title = "Teleport",
        Text = "Teleported to Frozen Island",
        Duration = 0
    })
end)

teleportTab:AddButton("Mythical Island", function()
    local player = game.Players.LocalPlayer
    local character = player.Character or player.CharacterAdded:Wait()
    local humanoidRootPart = character:WaitForChild("HumanoidRootPart")
    humanoidRootPart.CFrame = CFrame.new(2255, 7, 1071)
    game:GetService("StarterGui"):SetCore("SendNotification", {
        Title = "Teleport",
        Text = "Teleported to Mythical Island",
        Duration = 0
    })
end)

teleportTab:AddButton("****** Island", function()
    local player = game.Players.LocalPlayer
    local character = player.Character or player.CharacterAdded:Wait()
    local humanoidRootPart = character:WaitForChild("HumanoidRootPart")
    humanoidRootPart.CFrame = CFrame.new(-6768, 7, -1287)
    game:GetService("StarterGui"):SetCore("SendNotification", {
        Title = "Teleport",
        Text = "Teleported to ****** Island",
        Duration = 0
    })
end)

teleportTab:AddButton("Legend Island", function()
    local player = game.Players.LocalPlayer
    local character = player.Character or player.CharacterAdded:Wait()
    local humanoidRootPart = character:WaitForChild("HumanoidRootPart")
    humanoidRootPart.CFrame = CFrame.new(4604, 991, -3887)
    game:GetService("StarterGui"):SetCore("SendNotification", {
        Title = "Teleport",
        Text = "Teleported to Legend Island",
        Duration = 0
    })
end)

teleportTab:AddButton("Muscle King Island", function()
    local player = game.Players.LocalPlayer
    local character = player.Character or player.CharacterAdded:Wait()
    local humanoidRootPart = character:WaitForChild("HumanoidRootPart")
    humanoidRootPart.CFrame = CFrame.new(-8646, 17, -5738)
    game:GetService("StarterGui"):SetCore("SendNotification", {
        Title = "Teleport",
        Text = "Teleported to Muscle King",
        Duration = 0
    })
end)

teleportTab:AddButton("Jungle Island", function()
    local player = game.Players.LocalPlayer
    local character = player.Character or player.CharacterAdded:Wait()
    local humanoidRootPart = character:WaitForChild("HumanoidRootPart")
    humanoidRootPart.CFrame = CFrame.new(-8659, 6, 2384)
    game:GetService("StarterGui"):SetCore("SendNotification", {
        Title = "Teleport",
        Text = "Teleported to Jungle Island",
        Duration = 0
    })
end)

teleportTab:AddButton("Lava Brawl", function()
    local player = game.Players.LocalPlayer
    local character = player.Character or player.CharacterAdded:Wait()
    local humanoidRootPart = character:WaitForChild("HumanoidRootPart")
    humanoidRootPart.CFrame = CFrame.new(4471, 119, -8836)
    game:GetService("StarterGui"):SetCore("SendNotification", {
        Title = "Teleport",
        Text = "Teleported to Lava Brawl",
        Duration = 0
    })
end)

teleportTab:AddButton("Desert Brawl", function()
    local player = game.Players.LocalPlayer
    local character = player.Character or player.CharacterAdded:Wait()
    local humanoidRootPart = character:WaitForChild("HumanoidRootPart")
    humanoidRootPart.CFrame = CFrame.new(960, 17, -7398)
    game:GetService("StarterGui"):SetCore("SendNotification", {
        Title = "Teleport",
        Text = "Teleported to Desert Brawl",
        Duration = 0
    })
end)

teleportTab:AddButton("Regular Brawl", function()
    local player = game.Players.LocalPlayer
    local character = player.Character or player.CharacterAdded:Wait()
    local humanoidRootPart = character:WaitForChild("HumanoidRootPart")
    humanoidRootPart.CFrame = CFrame.new(-1849, 20, -6335)
    game:GetService("StarterGui"):SetCore("SendNotification", {
        Title = "Teleport",
        Text = "Teleported to Regular Brawl",
        Duration = 0
    })
end)

-- --- Crystals Tab ---
local crystalsTab = MainWindow:AddTab("Crystals")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

-- Crystal info
local crystalData = {
    ["Blue Crystal"] = {
        {name = "Blue Birdie", rarity = "Basic"},
        {name = "Orange Hedgehog", rarity = "Basic"},
        {name = "Blue Aura", rarity = "Basic"},
        {name = "Red Kitty", rarity = "Basic"},
        {name = "Dark Vampy", rarity = "Advanced"},
        {name = "Blue Bunny", rarity = "Basic"},
        {name = "Red Aura", rarity = "Basic"},
        {name = "Blue Aura", rarity = "Basic"},
        {name = "Green Aura", rarity = "Basic"},
        {name = "Purple Aura", rarity = "Basic"},
        {name = "Red Aura", rarity = "Basic"},
        {name = "Yellow Aura", rarity = "Basic"}
    },
    ["Green Crystal"] = {
        {name = "Silver Dog", rarity = "Basic"},
        {name = "Green Aura", rarity = "Advanced"},
        {name = "Dark Golem", rarity = "Advanced"},
        {name = "Green Butterfly", rarity = "Advanced"},
        {name = "Crimson Falcon", rarity = "Rare"},
        {name = "Red Aura", rarity = "Basic"},
        {name = "Blue Aura", rarity = "Basic"},
        {name = "Green Aura", rarity = "Basic"},
        {name = "Purple Aura", rarity = "Basic"},
        {name = "Red Aura", rarity = "Basic"},
        {name = "Yellow Aura", rarity = "Basic"}
    },
    ["Frost Crystal"] = {
        {name = "Yellow Butterfly", rarity = "Advanced"},
        {name = "Purple Dragon", rarity = "Rare"},
        {name = "Blue Phoenix", rarity = "Epic"},
        {name = "Orange Pegasus", rarity = "Rare"},
        {name = "Lightning", rarity = "Rare"},
        {name = "Electro", rarity = "Advanced"}
    },
    ["Mythical Crystal"] = {
        {name = "Purple Falcon", rarity = "Rare"},
        {name = "Red Dragon", rarity = "Rare"},
        {name = "Blue Firecaster", rarity = "Epic"},
        {name = "Golden Phoenix", rarity = "Epic"},
        {name = "Power Lightning", rarity = "Rare"},
        {name = "Dark Lightning", rarity = "Epic"}
    },
    ["Inferno Crystal"] = {
        {name = "Red Firecaster", rarity = "Epic"},
        {name = "Infernal Dragon", rarity = "Unique"},
        {name = "White Pegasus", rarity = "Rare"},
        {name = "Golden Phoenix", rarity = "Epic"},
        {name = "Inferno", rarity = "Epic"},
        {name = "Dark Storm", rarity = "Unique"}
    },
    ["Legends Crystal"] = {
        {name = "Ultra Birdie", rarity = "Unique"},
        {name = "Magic Butterfly", rarity = "Unique"},
        {name = "Green Firecaster", rarity = "Epic"},
        {name = "White Phoenix", rarity = "Epic"},
        {name = "Supernova", rarity = "Epic"},
        {name = "Purple Nova", rarity = "Unique"}
    },
    ["Muscle Elite Crystal"] = {
        {name = "Frostwave Legends Penguin", rarity = "Rare"},
        {name = "Phantom Genesis Dragon", rarity = "Rare"},
        {name = "Dark Legends Manticore", rarity = "Epic"},
        {name = "Ultimate Supernova Pegasus", rarity = "Epic"},
        {name = "Aether Spirit Bunny", rarity = "Unique"},
        {name = "Cybernetic Showdown Dragon", rarity = "Unique"}
    },
    ["Galaxy Oracle Crystal"] = {
        {name = "Eternal Strike Leviathan", rarity = "Rare"},
        {name = "Lightning Strike Phantom", rarity = "Epic"},
        {name = "Darkstar Hunter", rarity = "Unique"},
        {name = "Muscle King", rarity = "Unique"},
        {name = "Azure Tundra", rarity = "Epic"},
        {name = "Ultra Inferno", rarity = "Rare"}
    },
    ["Jungle Crystal"] = {
        {name = "Entropic Blast", rarity = "Unique"},
        {name = "Muscle Sensei", rarity = "Unique"},
        {name = "Grand Supernova", rarity = "Epic"},
        {name = "Neon Guardian", rarity = "Unique"},
        {name = "Eternal Megastrike", rarity = "Unique"},
        {name = "Golden Viking", rarity = "Epic"},
        {name = "Astral Electro", rarity = "Epic"},
        {name = "Dark Electro", rarity = "Epic"},
        {name = "Enchanted Mirage", rarity = "Epic"},
        {name = "Ultra Mirage", rarity = "Unique"},
        {name = "Unstable Mirage", rarity = "Unique"}
    }
}

-- Function to get all pets and auras
local function getAllPetsAndAuras()
    local allPets = {}
    local allAuras = {}

    for crystalName, pets in pairs(crystalData) do
        for _, pet in ipairs(pets) do
            if string.find(pet.name, "Aura") then
                if not allAuras[pet.name] then
                    allAuras[pet.name] = {name = pet.name, rarity = pet.rarity, crystal = crystalName}
                end
            else
                if not allPets[pet.name] then
                    allPets[pet.name] = {name = pet.name, rarity = pet.rarity, crystal = crystalName}
                end
            end
        end
    end

    return allPets, allAuras
end

-- Find which crystal contains an item
local function findCrystalForItem(itemName)
    for crystalName, pets in pairs(crystalData) do
        for _, pet in ipairs(pets) do
            if pet.name == itemName then
                return crystalName
            end
        end
    end
    return nil
end

-- Track current selections
local selectedPet = ""
local selectedAura = ""

local allPets, allAuras = getAllPetsAndAuras()

-- --- Pet Dropdown ---
local petDropdown = crystalsTab:AddDropdown("Select Pet", function(text)
    selectedPet = text
    local crystal = findCrystalForItem(text)
    print("Selected Pet: " .. text .. " (Found in: " .. (crystal or "Unknown") .. ")")
end)

-- Manually add all pets (sorted by rarity)
-- Basic Pets
petDropdown:Add("Blue Birdie (Basic)")
petDropdown:Add("Orange Hedgehog (Basic)")
petDropdown:Add("Red Kitty (Basic)")
petDropdown:Add("Blue Bunny (Basic)")
petDropdown:Add("Silver Dog (Basic)")

-- Advanced Pets
petDropdown:Add("Dark Vampy (Advanced)")
petDropdown:Add("Dark Golem (Advanced)")
petDropdown:Add("Green Butterfly (Advanced)")
petDropdown:Add("Yellow Butterfly (Advanced)")

-- Rare Pets
petDropdown:Add("Crimson Falcon (Rare)")
petDropdown:Add("Purple Dragon (Rare)")
petDropdown:Add("Orange Pegasus (Rare)")
petDropdown:Add("Purple Falcon (Rare)")
petDropdown:Add("Red Dragon (Rare)")
petDropdown:Add("White Pegasus (Rare)")
petDropdown:Add("Frostwave Legends Penguin (Rare)")
petDropdown:Add("Phantom Genesis Dragon (Rare)")
petDropdown:Add("Eternal Strike Leviathan (Rare)")

-- Epic Pets
petDropdown:Add("Blue Pheonix (Epic)")
petDropdown:Add("Blue Firecaster (Epic)")
petDropdown:Add("Golden Pheonix (Epic)")
petDropdown:Add("Red Firecaster (Epic)")
petDropdown:Add("Green Firecaster (Epic)")
petDropdown:Add("White Pheonix (Epic)")
petDropdown:Add("Dark Legends Manticore (Epic)")
petDropdown:Add("Ultimate Supernova Pegasus (Epic)")
petDropdown:Add("Lightning Strike Phantom (Epic)")
petDropdown:Add("Golden Viking (Epic)")

-- Unique Pets
petDropdown:Add("Infernal Dragon (Unique)")
petDropdown:Add("Ultra Birdie (Unique)")
petDropdown:Add("Magic Butterfly (Unique)")
petDropdown:Add("Aether Spirit Bunny (Unique)")
petDropdown:Add("Cybernetic Showdown Dragon (Unique)")
petDropdown:Add("Darkstar Hunter (Unique)")
petDropdown:Add("Muscle Sensei (Unique)")
petDropdown:Add("Neon Guardian (Unique)")

-- --- Aura Dropdown ---
local auraDropdown = crystalsTab:AddDropdown("Select Aura", function(text)
    selectedAura = text
    local crystal = findCrystalForItem(text)
    print("Selected Aura: " .. text .. " (Found in: " .. (crystal or "Unknown") .. ")")
end)

-- Manually add all auras (sorted by rarity)
auraDropdown:Add("Blue Aura (Basic)")
auraDropdown:Add("Green Aura (Basic)")
auraDropdown:Add("Purple Aura (Basic)")
auraDropdown:Add("Red Aura (Basic)")
auraDropdown:Add("Yellow Aura (Basic)")
auraDropdown:Add("Ultra Inferno  (Rare)")
auraDropdown:Add("Azure Tundra (Epic)")
auraDropdown:Add("Grand Supernova (Epic)")
auraDropdown:Add("Muscle King (Unique)")
auraDropdown:Add("Entropic Blast (Unique)")
auraDropdown:Add("Eternal Megastrike (Unique)")

crystalsTab:AddLabel("=== System to buy ===")

-- --- Auto buy pet ---
crystalsTab:AddSwitch("Auto Buy Pet", function(isActive)
    _G.AutoBuyPet = isActive
    if isActive then
        if selectedPet == "" then
            print("Please select a pet first!")
            return
        end
        -- Remove rarity from selection
        local petName = selectedPet:match("^(.-)%s*%(")
        if not petName then petName = selectedPet end
        local crystal = findCrystalForItem(petName)
        if not crystal then
            print("Could not find crystal for pet: " .. petName)
            return
        end
        print("Auto buying pet: " .. petName .. " from " .. crystal)
        spawn(function()
            while _G.AutoBuyPet and selectedPet ~= "" do
                local petToBuy = ReplicatedStorage.cPetShopFolder:FindFirstChild(petName)
                if petToBuy then
                    ReplicatedStorage.cPetShopRemote:InvokeServer(petToBuy)
                    print("Bought pet: " .. petName)
                else
                    print("Pet not found: " .. petName)
                end
                task.wait(0.1)
            end
        end)
    else
        print("Auto buy pet stopped")
    end
end)

-- --- Auto buy aura ---
crystalsTab:AddSwitch("Auto buy Aura", function(isActive)
    _G.AutoBuyAura = isActive
    if isActive then
        if selectedAura == "" then
            print("Please select an aura first!")
            return
        end
        -- Remove rarity from selection
        local auraName = selectedAura:match("^(.-)%s*%(")
        if not auraName then auraName = selectedAura end
        local crystal = findCrystalForItem(auraName)
        if not crystal then
            print("Could not find crystal for aura: " .. auraName)
            return
        end
        print("Auto buying aura: " .. auraName .. " from " .. crystal)
        spawn(function()
            while _G.AutoBuyAura and selectedAura ~= "" do
                local auraToBuy = ReplicatedStorage.cPetShopFolder:FindFirstChild(auraName)
                if auraToBuy then
                    ReplicatedStorage.cPetShopRemote:InvokeServer(auraToBuy)
                    print("Bought aura: " .. auraName)
                else
                    print("Aura not found: " .. auraName)
                end
                task.wait(0.1)
            end
        end)
    else
        print("Auto buy aura stopped")
    end
end)

-- --- Gift System ---
local giftTab = MainWindow:AddTab("Gift")
giftTab:AddLabel("Gifting Protein Egg:").TextSize = 22

local proteinEggLabel = giftTab:AddLabel("Protein Eggs: 0")
proteinEggLabel.TextSize = 20

local selectedEggPlayer = nil
local eggCount = 0

local eggDropdown = giftTab:AddDropdown("Player to Gift Eggs", function(selectedDisplayName)
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr.DisplayName == selectedDisplayName then
            selectedEggPlayer = plr
            break
        end
    end
end)

for _, plr in ipairs(Players:GetPlayers()) do
    if plr ~= Players.LocalPlayer then
        eggDropdown:Add(plr.DisplayName)
    end
end

Players.PlayerAdded:Connect(function(plr)
    if plr ~= Players.LocalPlayer then
        eggDropdown:Add(plr.DisplayName)
    end
end)

giftTab:AddTextBox("Amount of Eggs", function(text)
    eggCount = tonumber(text) or 0
end)

giftTab:AddButton("Gift Eggs", function()
    if selectedEggPlayer and eggCount > 0 then
        for i = 1, eggCount do
            local egg = Players.LocalPlayer.consumablesFolder:FindFirstChild("Protein Egg")
            if egg then
                ReplicatedStorage.rEvents.giftRemote:InvokeServer("giftRequest", selectedEggPlayer, egg)
                task.wait(0.1)
            end
        end
    end
end)

-- --- Tropical Shakes Gift ---
giftTab:AddLabel("Gifting Tropical Shakes:").TextSize = 22

local tropicalShakeLabel = giftTab:AddLabel("Tropical Shakes: 0")
tropicalShakeLabel.TextSize = 18

local selectedShakePlayer = nil
local shakeCount = 0

local shakeDropdown = giftTab:AddDropdown("Player to Gift Tropical Shakes", function(selectedDisplayName)
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr.DisplayName == selectedDisplayName then
            selectedShakePlayer = plr
            break
        end
    end
end)

for _, plr in ipairs(Players:GetPlayers()) do
    if plr ~= Players.LocalPlayer then
        shakeDropdown:Add(plr.DisplayName)
    end
end

Players.PlayerAdded:Connect(function(plr)
    if plr ~= Players.LocalPlayer then
        shakeDropdown:Add(plr.DisplayName)
    end
end)

giftTab:AddTextBox("Tropical Shakes gift", function(text)
    shakeCount = tonumber(text) or 0
end)

giftTab:AddButton("Gift Tropical Shakes", function()
    if selectedShakePlayer and shakeCount > 0 then
        for i = 1, shakeCount do
            local shake = Players.LocalPlayer.consumablesFolder:FindFirstChild("Tropical Shake")
            if shake then
                ReplicatedStorage.rEvents.giftRemote:InvokeServer("giftRequest", selectedShakePlayer, shake)
                task.wait(0.1)
            end
        end
    end
end)

-- --- Item Count Update ---
local function updateItemCounts()
    local proteinEggCount = 0
    local tropicalShakeCount = 0

    local backpack = Players.LocalPlayer:WaitForChild("Backpack")
    if backpack then
        for _, item in ipairs(backpack:GetChildren()) do
            if item.Name == "Protein Egg" then
                proteinEggCount = proteinEggCount + 1
            elseif item.Name == "Tropical Shake" or item.Name == "Piñas" then
                tropicalShakeCount = tropicalShakeCount + 1
            end
        end
    end

    proteinEggLabel:Set("Protein Eggs: " .. proteinEggCount)
    tropicalShakeLabel:Set("Tropical Shakes: " .. tropicalShakeCount)
end

task.spawn(function()
    while true do
        updateItemCounts()
        task.wait(0.25)
    end
end)

-- --- Auto Eat Boosts ---
local autoEatBoosts = false
local boostList = {
    "ULTRA Shake",
    "TOUGH Bar",
    "Protein Shake",
    "Energy Shake",
    "Protein Bar",
    "Energy Bar",
    "Tropical Shake"
}

local function eatAllBoosts()
    local player = game.Players.LocalPlayer
    local backpack = player:WaitForChild("Backpack")
    local character = player.Character or player.CharacterAdded:Wait()

    for _, boostName in ipairs(boostList) do
        local boost = backpack:FindFirstChild(boostName)
        while boost do
            boost.Parent = character
            pcall(function()
                boost:Activate()
            end)
            task.wait(0)
            boost = backpack:FindFirstChild(boostName)
        end
    end
end

task.spawn(function()
    while true do
        if autoEatBoosts then
            eatAllBoosts()
            task.wait(2)
        else
            task.wait(1)
        end
    end
end)

-- --- Auto Clear Inventory ---
giftTab:AddSwitch("Auto Clear Inventory", function(state)
    autoEatBoosts = state
end)

-- End of script
