local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local Lighting = game:GetService("Lighting")
local Workspace = game:GetService("Workspace")
local LocalPlayer = Players.LocalPlayer

-- Create main GUI
local gui = Instance.new("ScreenGui", LocalPlayer:WaitForChild("PlayerGui"))
gui.Name = "MyFeatureUI"

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 520, 0, 600)
frame.Position = UDim2.new(0.5, -260, 0.5, -300)
frame.BackgroundColor3 = Color3.fromRGB(0, 10, 0)
frame.BorderSizePixel = 0

-- Make draggable function
local function makeDraggable(f)
    local dragging = false
    local dragStart, startPos
    f.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            dragStart = input.Position
            startPos = f.Position
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
            f.Position = UDim2.new(
                startPos.X.Scale,
                startPos.X.Offset + delta.X,
                startPos.Y.Scale,
                startPos.Y.Offset + delta.Y
            )
        end
    end)
end
makeDraggable(frame)

-- Placeholder functions for GUI components
local function AddDropdown(label, callback)
    -- Implement your dropdown UI here
    local dropdown = {}
    function dropdown:Add(text)
        -- Add option to dropdown
    end
    function dropdown:Clear()
        -- Clear options
    end
    return dropdown
end

local function AddSwitch(label, callback)
    -- Implement your switch UI here
end

local function AddButton(label, callback)
    -- Implement your button UI here
end

-- Pet selection dropdown
local petDropdown = AddDropdown("Select Pet", function(selectedPetName)
    local petsFolder = LocalPlayer:FindFirstChild("petsFolder")
    if not petsFolder then return end

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
    local petsToEquip = {}
    for _, pet in pairs(petsFolder.Unique:GetChildren()) do
        if pet.Name == selectedPetName then
            table.insert(petsToEquip, pet)
        end
    end

    local maxPets = 8
    for i = 1, math.min(#petsToEquip, maxPets) do
        game:GetService("ReplicatedStorage").rEvents.equipPetEvent:FireServer("equipPet", petsToEquip[i])
        task.wait(0.1)
    end
end)

local wildWizardOption = petDropdown:Add("Wild Wizard")
local mightyMonsterOption = petDropdown:Add("Mighty Monster")

-- Auto "Good Karma" switch
local autoGoodKarma = false
AddSwitch("Auto Good Karma", function(isActive)
    autoGoodKarma = isActive
    task.spawn(function()
        while autoGoodKarma do
            local character = LocalPlayer.Character
            local rightHand = character and character:FindFirstChild("RightHand")
            local leftHand = character and character:FindFirstChild("LeftHand")
            if rightHand and leftHand then
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

-- Auto "Bad Karma" switch
local autoBadKarma = false
AddSwitch("Auto Bad Karma", function(isActive)
    autoBadKarma = isActive
    task.spawn(function()
        while autoBadKarma do
            local character = LocalPlayer.Character
            local rightHand = character and character:FindFirstChild("RightHand")
            local leftHand = character and character:FindFirstChild("LeftHand")
            if rightHand and leftHand then
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

-- Whitelist friends auto-adding
local playerWhitelist = {}
local friendWhitelistActive = false
AddSwitch("Auto Whitelist Friends", function(state)
    friendWhitelistActive = state
    if state then
        for _, player in ipairs(Players:GetPlayers()) do
            if player ~= LocalPlayer and LocalPlayer:IsFriendsWith(player.UserId) then
                playerWhitelist[player.Name] = true
            end
        end
        Players.PlayerAdded:Connect(function(player)
            if player ~= LocalPlayer and LocalPlayer:IsFriendsWith(player.UserId) then
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

AddTextBox("Whitelist", function(text)
    local target = Players:FindFirstChild(text)
    if target then
        playerWhitelist[target.Name] = true
    end
end)

AddTextBox("UnWhitelist", function(text)
    local target = Players:FindFirstChild(text)
    if target then
        playerWhitelist[target.Name] = nil
    end
end)

-- Auto Kill switch
local autoKill = false
AddSwitch("Auto Kill", function(isActive)
    autoKill = isActive
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

-- Target list for kill
local targetPlayerNames = {}
local selectedTarget = nil

local targetDropdown = AddDropdown("Select Target", function(displayName)
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

local function refreshTargetDropdown()
    targetDropdown:Clear()
    for _, name in ipairs(targetPlayerNames) do
        local player = Players:FindFirstChild(name)
        if player then
            targetDropdown:Add(player.DisplayName)
        end
    end
end

-- Initialize target list
for _, player in ipairs(Players:GetPlayers()) do
    if player ~= LocalPlayer then
        targetDropdown:Add(player.DisplayName)
        table.insert(targetPlayerNames, player.Name)
    end
end

Players.PlayerAdded:Connect(function(player)
    if player ~= LocalPlayer then
        targetDropdown:Add(player.DisplayName)
        table.insert(targetPlayerNames, player.Name)
    end
end)

Players.PlayerRemoving:Connect(function(player)
    for i = #targetPlayerNames, 1, -1 do
        if targetPlayerNames[i] == player.Name then
            table.remove(targetPlayerNames, i)
        end
    end
    refreshTargetDropdown()
    if selectedTarget == player.Name then
        selectedTarget = nil
    end
end)

-- Remove selected target
AddButton("Remove Selected Target", function()
    if selectedTarget then
        for i, v in ipairs(targetPlayerNames) do
            if v == selectedTarget then
                table.remove(targetPlayerNames, i)
                break
            end
        end
        selectedTarget = nil
        refreshTargetDropdown()
    end
end)

-- Kill target loop
local killTarget = false
AddSwitch("Start Kill Target", function(state)
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

-- View player (spy)
local spyTargetPlayerName = nil
local spyDropdown = AddDropdown("Select View Target", function(displayName)
    for _, player in ipairs(Players:GetPlayers()) do
        if player.DisplayName == displayName then
            spyTargetPlayerName = player.Name
            break
        end
    end
end)

for _, player in ipairs(Players:GetPlayers()) do
    if player ~= LocalPlayer then
        spyDropdown:Add(player.DisplayName)
    end
end

local spying = false
AddSwitch("View Player", function(isActive)
    spying = isActive
    if not spying then
        local cam = workspace.CurrentCamera
        cam.CameraSubject = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") or LocalPlayer
        return
    end
    task.spawn(function()
        while spying do
            local target = Players:FindFirstChild(spyTargetPlayerName)
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

-- Remove punch animation
local function RecoveryPunch()
    if _G.AnimBlockConnection then
        _G.AnimBlockConnection:Disconnect()
        _G.AnimBlockConnection = nil
    end
    if _G.AnimMonitorConnection then
        _G.AnimMonitorConnection:Disconnect()
        _G.AnimMonitorConnection = nil
    end
    if _G.ToolConnections then
        for _, conn in pairs(_G.ToolConnections) do
            if conn then conn:Disconnect() end
        end
        _G.ToolConnections = nil
    end
    if _G.BackpackAddedConnection then
        _G.BackpackAddedConnection:Disconnect()
        _G.BackpackAddedConnection = nil
    end
    if _G.CharacterToolAddedConnection then
        _G.CharacterToolAddedConnection:Disconnect()
        _G.CharacterToolAddedConnection = nil
    end
    if _G.CharacterAddedConnection then
        _G.CharacterAddedConnection:Disconnect()
        _G.CharacterAddedConnection = nil
    end
end

AddButton("Recover Punch Anim", function()
    RecoveryPunch()
end)

-- Auto equip punch
local autoEquipPunch = false
AddSwitch("Auto Equip Punch", function(state)
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

-- Auto punch without animation
local autoPunchNoAnim = false
AddSwitch("Auto Punch [without animation]", function(state)
    autoPunchNoAnim = state
    task.spawn(function()
        while autoPunchNoAnim do
            local punch = LocalPlayer.Backpack:FindFirstChild("Punch") or (LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Punch"))
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

-- Auto punch
local _G = _G or {}
local function setAutoPunch(state)
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
                task.wait(0.1)
            end
        end)
        task.spawn(function()
            while _G.autoPunchActive do
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
end

AddSwitch("Auto Punch", function(state)
    setAutoPunch(state)
end)

-- Fast punch
local _G = _G or {}
local autoFastPunch = false
AddSwitch("Fast Punch", function(state)
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
            end
        end)
        task.spawn(function()
            while _G.autoPunchActive do
                local punch = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Punch")
                if punch then
                    punch:Activate()
                end
                task.wait()
            end
        end)
    else
        local punch = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Punch")
        if punch then
            punch.Parent = LocalPlayer.Backpack
        end
    end
end)

-- God Mode toggle
local godMode = false
AddSwitch("God Mode", function(state)
    godMode = state
    if state then
        task.spawn(function()
            while godMode do
                game:GetService("ReplicatedStorage").rEvents.brawlEvent:FireServer("joinBrawl")
                task.wait()
            end
        end)
    end
end)

-- Follow system
local following = false
local followTarget = nil

local function followPlayer(targetPlayer)
    local myChar = LocalPlayer.Character
    local targetChar = targetPlayer.Character
    if not (myChar and targetChar) then return end
    local myHRP = myChar:FindFirstChild("HumanoidRootPart")
    local targetHRP = targetChar:FindFirstChild("HumanoidRootPart")
    if myHRP and targetHRP then
        local followPos = targetHRP.CFrame * CFrame.new(0, 0, -3).Position
        myHRP.CFrame = CFrame.new(followPos, targetHRP.Position)
    end
end

local followDropdown = AddDropdown("Follow Player", function(selectedDisplayName)
    if selectedDisplayName and selectedDisplayName ~= "" then
        local target = nil
        for _, plr in ipairs(Players:GetPlayers()) do
            if plr.DisplayName == selectedDisplayName then
                target = plr
                break
            end
        end
        if target then
            followTarget = target.Name
            following = true
            print("✅ Started following: " .. target.Name)
            followPlayer(target)
        end
    end
end)

-- Populate follow list
local function updateFollowList()
    followDropdown:Clear()
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer then
            followDropdown:Add(player.DisplayName)
        end
    end
end

for _, player in ipairs(Players:GetPlayers()) do
    if player ~= LocalPlayer then
        followDropdown:Add(player.DisplayName)
    end
end

Players.PlayerAdded:Connect(function(player)
    if player ~= LocalPlayer then
        followDropdown:Add(player.DisplayName)
        updateFollowList()
    end
end)

Players.PlayerRemoving:Connect(function(player)
    updateFollowList()
    if followTarget == player.Name then
        followTarget = nil
        following = false
    end
end)

AddButton("Stop Following", function()
    following = false
    followTarget = nil
    print("⛔ Stopped following")
end)

task.spawn(function()
    while true do
        if following and followTarget then
            local target = Players:FindFirstChild(followTarget)
            if target then
                followPlayer(target)
            else
                following = false
                followTarget = nil
            end
        end
        task.wait(0.01)
    end
end)

LocalPlayer.CharacterAdded:Connect(function()
    task.wait(1)
    if following and followTarget then
        local target = Players:FindFirstChild(followTarget)
        if target then
            followPlayer(target)
        end
    end
end)

-- Auto Slam (auto ground pound)
local autoSlam = false
AddSwitch("auto slams", function(state)
    autoSlam = state
    if state then
        task.spawn(function()
            while autoSlam do
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

-- Execute scripts from URLs
local scriptUrls = {
    "https://raw.githubusercontent.com/SadOz8/Stuffs/refs/heads/main/Crack",
    "https://raw.githubusercontent.com/SadOz8/Stuffs/refs/heads/main/Crack2",
    "https://raw.githubusercontent.com/SadOz8/Stuffs/refs/heads/main/Crack4",
    "https://raw.githubusercontent.com/SadOz8/Stuffs/refs/heads/main/Crack5",
    "https://raw.githubusercontent.com/SadOz8/Stuffs/refs/heads/main/Crack6"
}

AddButton("Execute Scripts", function()
    for _, url in ipairs(scriptUrls) do
        spawn(function()
            local success, response = pcall(function()
                return game:HttpGet(url)
            end)
            if success and response then
                local loadSuccess, err = pcall(function()
                    loadstring(response)()
                end)
                if not loadSuccess then
                    warn("[Error executing script]:", url, err)
                end
            else
                warn("[Failed to load script]:", url)
            end
        end)
    end
end)

-- Change time of day
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

local timeDropdown = AddDropdown("Change Time", function(selection)
    -- Reset lighting
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

-- Add options to dropdown
for _, option in ipairs(timeOptions) do
    timeDropdown:Add(option)
end
