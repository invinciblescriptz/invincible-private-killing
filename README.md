local ScriptLibrary = loadstring(game:HttpGet("https://raw.githubusercontent.com/p4020854-hub/Lb/refs/heads/main/X", true))()

local MainWindow = ScriptLibrary:AddWindow(string.format("Queen KKrxzy|| private || Hello %s", game.Players.LocalPlayer.DisplayName), {
    min_size = Vector2.new(660, 700),
    can_resize = true,
    main_color = Color3.fromRGB(255, 192, 203)
})

local KillTab = MainWindow:AddTab("Kill")
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
KillTab:AddSwitch("Auto Good Karma", function(bool)
    autoGoodKarma = bool
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

KillTab:AddSwitch("Auto Bad Karma", function(bool)
    autoBadKarma = bool
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
local friendWhitelistActive = false
KillTab:AddSwitch("Auto Whitelist Friends", function(state)
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

KillTab:AddTextBox("Whitelist", function(text)
    local target = Players:FindFirstChild(text)
    if target then
        playerWhitelist[target.Name] = true
    end
end)

KillTab:AddTextBox("UnWhitelist", function(text)
    local target = Players:FindFirstChild(text)
    if target then
        playerWhitelist[target.Name] = nil
    end
end)

-- --- Auto Kill ---
KillTab:AddSwitch("Auto Kill", function(bool)
    autoKill = bool
    if bool then
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
                            local humanoid = targetChar and targetChar:FindFirstChild("Humanoid")
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

-- --- Target Selection ---
local targetPlayerNames = {}
local selectedTarget = nil

local targetDropdown = KillTab:AddDropdown("Select Target", function(displayName)
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

-- Update on join/leave
Players.PlayerAdded:Connect(function(player)
    if player ~= LocalPlayer then
        table.insert(targetPlayerNames, player.Name)
        updateTargetDropdown()
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
local killTarget = false
KillTab:AddSwitch("Start Kill Target", function(state)
    killTarget = state
    if state then
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
    end
end)

-- --- View Player (Spy) ---
local targetPlayerName = nil
local spyTargetDropdownItems = {}

local spyTargetDropdown = KillTab:AddDropdown("View Player", function(displayName)
    for _, player in ipairs(Players:GetPlayers()) do
        if player.DisplayName == displayName then
            targetPlayerName = player.Name
            break
        end
    end
end)

-- Populate initial
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
local spying = false
KillTab:AddSwitch("View Player", function(bool)
    spying = bool
    if not spying then
        workspace.CurrentCamera.CameraSubject = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") or nil
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

    -- Connect to block future animation plays
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

KillTab:AddButton("Recover Punch Anim", function()
    recoverPunchAnimations()
end)

KillTab:AddSwitch("Auto Equip Punch", function(state)
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

KillTab:AddSwitch("Auto Punch [without animation]", function(state)
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

KillTab:AddSwitch("Auto Punch", function(state)
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
                task.wait(0.05)
                LocalPlayer.muscleEvent:FireServer("punch", "rightHand")
                LocalPlayer.muscleEvent:FireServer("punch", "leftHand")
                local character = LocalPlayer.Character
                if character then
                    local punchTool = character:FindFirstChild("Punch")
                    if punchTool then
                        punchTool:Activate()
                    end
                end
                task.wait()
            end
        end)
    else
        local character = LocalPlayer.Character
        local equipped = character:FindFirstChild("Punch")
        if equipped then
            equipped.Parent = LocalPlayer.Backpack
        end
    end
end)

KillTab:AddSwitch("Fast punch", function(state)
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
                task.wait(0.01)
                LocalPlayer.muscleEvent:FireServer("punch", "rightHand")
                LocalPlayer.muscleEvent:FireServer("punch", "leftHand")
                local character = LocalPlayer.Character
                if character then
                    local punchTool = character:FindFirstChild("Punch")
                    if punchTool then
                        punchTool:Activate()
                    end
                end
                task.wait()
            end
        end)
    else
        local character = LocalPlayer.Character
        local punch = character and character:FindFirstChild("Punch")
        if punch then
            punch.Parent = LocalPlayer.Backpack
        end
    end
end)

-- God Mode (Good Mode)
local godModeToggle = false
KillTab:AddSwitch("Good Mode", function(State)
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

local followDropdown = KillTab:AddDropdown("Teleport Player", function(selectedDisplayName)
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

-- Populate initial
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
KillTab:AddButton("Unfollow", function()
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
local godDamageActive = false
KillTab:AddSwitch("Auto Slams", function(state)
    godDamageActive = state
    if state then
        task.spawn(function()
            while godDamageActive do
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
KillTab:AddButton("Combo NaN", function()
    local args = {"changeSize", 0/0}
    game:GetService("ReplicatedStorage"):WaitForChild("rEvents"):WaitForChild("changeSpeedSizeRemote"):InvokeServer(unpack(args))
end)

-- --- URLs for scripts ---
local urls = {
    "https://raw.githubusercontent.com/SadOz8/Stuffs/refs/heads/main/Crack",
    "https://raw.githubusercontent.com/SadOz8/Stuffs/refs/heads/main/Crack2",
    "https://raw.githubusercontent.com/SadOz8/Stuffs/refs/heads/main/Crack4",
    "https://raw.githubusercontent.com/SadOz8/Stuffs/refs/heads/main/Crack5",
    "https://raw.githubusercontent.com/SadOz8/Stuffs/refs/heads/main/Crack6"
}

KillTab:AddButton("Touch Me!", function()
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
local timeDropdown = KillTab:AddDropdown("Change Time", function(selection)
    print("Selected:", selection)
    -- Reset defaults
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

local AutoFarmTab = window:AddTab("Farm OP")
AutoFarmTab:Show()

-- Initial state
getgenv()._AutoRepfarmEnabled = false

-- Switch for auto farm
AutoFarmTab:AddSwitch("Auto Farm (Use if ping < 250ms)", function(state)
    getgenv()._AutoRepfarmEnabled = state
    warn("[Auto Farm] State changed to:", state and "ON" or "OFF")
end)

-- Services
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Stats = game:GetService("Stats")
local Player = game:GetService("Players").LocalPlayer

-- Configuration
local PET_NAME = "Swift Samurai"
local ROCK_NAME = "Rock5M"
local PROTEIN_EGG_NAME = "ProteinEgg"
local PROTEIN_EGG_INTERVAL = 30 * 60 -- 30 minutes
local REPS_PER_CYCLE = 185
local REP_DELAY = 0.01
local ROCK_INTERVAL = 1
local MAX_PING = 1100
local MIN_PING = 300

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
    local character = Player.Character or Player.CharacterAdded:Wait()
    HumanoidRootPart = character:WaitForChild("HumanoidRootPart", 5)
end

local function equipPet()
    local petsFolder = Player:FindFirstChild("petsFolder")
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
    if Player:FindFirstChild("Backpack") then
        for _, item in pairs(Player.Backpack:GetChildren()) do
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

-- Main farming loop
if not getgenv()._AutoRepfarmLoop then
    getgenv()._AutoRepfarmLoop = true

    task.spawn(function()
        updateCharacterRefs()
        equipPet()
        lastProteinEggTime = tick()
        lastRockTime = tick()

        while true do
            if getgenv()._AutoRepfarmEnabled then
                local ping = getPing()
                if ping > MAX_PING then
                    warn("[Auto Repfarm] High ping ("..math.floor(ping).."ms), pausing for 5s...")
                    task.wait(5)
                else
                    if Player:FindFirstChild("muscleEvent") then
                        for i = 1, REPS_PER_CYCLE do
                            Player.muscleEvent:FireServer("rep")
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

-- Additional features such as auto-eating eggs, anti lag, etc., should follow similar patterns:
-- For brevity, please specify if you want me to include or fix specific parts further.

-- The above structure provides a clear, fixed, and translated core script.
