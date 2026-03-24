-- Services
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Lighting = game:GetService("Lighting")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local LocalPlayer = Players.LocalPlayer

-- Remote Event
local muscleEvent = LocalPlayer:FindFirstChild("muscleEvent") or ReplicatedStorage:FindFirstChild("muscleEvent")
if not muscleEvent then
    warn("muscleEvent not found! Make sure it exists.")
end

-- UI Library
local WindUI = loadstring(game:HttpGet("https://raw.githubusercontent.com/xHeptc/Kavo-UI-Library/main/source.lua"))()
local Window = WindUI.new({ Title = "Muscle Legends OP", Center = true, AutoShow = true })

-- Main Tab
local KillTab = Window:CreateTab("Kill")

-- Flags
local autoGoodKarma = false
local autoBadKarma = false
local autoKillAll = false
local autoTargetKill = false
local autoPunchNoAnim = false
local autoEquipPunch = false
local autoGroundSlams = false
local autoBrawl = false

local following = false
local followTargetName = nil

local playerWhitelist = {}
local targetPlayerNames = {}

-- Helper Functions
local function safeFire(event, ...)
    if event then
        pcall(function() event:FireServer(...) end)
    end
end

local function getHRPAndHum(plr)
    if plr and plr.Character then
        local hrp = plr.Character:FindFirstChild("HumanoidRootPart")
        local hum = plr.Character:FindFirstChild("Humanoid")
        return hrp, hum
    end
    return nil, nil
end

-- Pets Section
local PetsSection = KillTab:AddSection("Pets")
PetsSection:AddLabel("Select Pet (Damage / Durability)")
local petDropdown = PetsSection:AddDropdown("Select Pet", function(petName)
    -- Unequip all pets
    pcall(function()
        for _, folder in pairs(LocalPlayer.petsFolder:GetChildren()) do
            if folder:IsA("Folder") then
                for _, pet in pairs(folder:GetChildren()) do
                    ReplicatedStorage.rEvents.equipPetEvent:FireServer("unequipPet", pet)
                end
            end
        end
    end)
    task.wait(0.4)
    -- Equip selected pet(s) up to 8
    local count = 0
    for _, pet in pairs(LocalPlayer.petsFolder:FindFirstChild("Unique"):GetChildren()) do
        if pet.Name == petName and count < 8 then
            ReplicatedStorage.rEvents.equipPetEvent:FireServer("equipPet", pet)
            count = count + 1
            task.wait(0.12)
        end
    end
end)
petDropdown:Add("Wild Wizard")
petDropdown:Add("Mighty Monster")

-- Karma Auto Attack
local function karmaLoop(isGood)
    task.spawn(function()
        while (isGood and autoGoodKarma) or (not isGood and autoBadKarma) do
            local char = LocalPlayer.Character
            if char then
                for _, plr in ipairs(Players:GetPlayers()) do
                    if plr ~= LocalPlayer then
                        local evil = plr:FindFirstChild("evilKarma")
                        local good = plr:FindFirstChild("goodKarma")
                        if evil and good then
                            local condition = isGood and (evil.Value > good.Value) or (good.Value > evil.Value)
                            if condition then
                                local root, _ = getHRPAndHum(plr)
                                if root then
                                    local rh = char:FindFirstChild("RightHand")
                                    local lh = char:FindFirstChild("LeftHand")
                                    if rh and lh then
                                        firetouchinterest(rh, root, 0)
                                        firetouchinterest(lh, root, 0)
                                        safeFire(muscleEvent, "punch", "rightHand")
                                    end
                                end
                            end
                        end
                    end
                end
            end
            task.wait(0.03)
        end
    end)
end

KillTab:AddSwitch("Auto Good Karma", function(state)
    autoGoodKarma = state
    if state then karmaLoop(true) end
end)

KillTab:AddSwitch("Auto Bad Karma", function(state)
    autoBadKarma = state
    if state then karmaLoop(false) end
end)

-- Whitelist
local function toggleWhitelist(state)
    if state then
        for _, plr in ipairs(Players:GetPlayers()) do
            if plr ~= LocalPlayer and LocalPlayer:IsFriendsWith(plr.UserId) then
                playerWhitelist[plr.Name] = true
            end
        end
    else
        playerWhitelist = {}
    end
end

local WhitelistSection = KillTab:AddSection("Whitelist")
WhitelistSection:AddSwitch("Auto Whitelist Friends", toggleWhitelist)
WhitelistSection:AddTextBox("Whitelist Player", function(txt)
    local plr = Players:FindFirstChild(txt)
    if plr then playerWhitelist[plr.Name] = true end
end)
WhitelistSection:AddTextBox("Unwhitelist Player", function(txt)
    playerWhitelist[txt] = nil
end)

-- Auto Kill All
local killAllActive = false
KillTab:AddSwitch("Auto Kill All", function(state)
    killAllActive = state
    if not state then return end
    task.spawn(function()
        while killAllActive do
            local char = LocalPlayer.Character
            if char then
                for _, plr in ipairs(Players:GetPlayers()) do
                    if plr ~= LocalPlayer and not playerWhitelist[plr.Name] then
                        local root, _ = getHRPAndHum(plr)
                        if root then
                            safeFire(muscleEvent, "punch", "rightHand")
                            safeFire(muscleEvent, "punch", "leftHand")
                        end
                    end
                end
            end
            task.wait(0.02)
        end
    end)
end)

-- Target Kill
local targetPlayerNames = {}
local TargetSection = KillTab:AddSection("Targets")
local targetDropdown = TargetSection:AddDropdown("Add Kill Target", function(displayName)
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr.DisplayName == displayName and not table.find(targetPlayerNames, plr.Name) then
            table.insert(targetPlayerNames, plr.Name)
            break
        end
    end
end)
TargetSection:AddButton("Clear All Targets", function()
    targetPlayerNames = {}
end)
-- Populate dropdown
for _, plr in ipairs(Players:GetPlayers()) do
    if plr ~= LocalPlayer then
        targetDropdown:Add(plr.DisplayName)
    end
end)
Players.PlayerAdded:Connect(function(plr)
    if plr ~= LocalPlayer then
        targetDropdown:Add(plr.DisplayName)
    end
end)

local targetKillActive = false
KillTab:AddSwitch("Kill Selected Targets", function(state)
    targetKillActive = state
    if not state then return end
    task.spawn(function()
        while targetKillActive do
            for _, name in ipairs(targetPlayerNames) do
                local plr = Players:FindFirstChild(name)
                if plr and plr.Character then
                    local root, _ = getHRPAndHum(plr)
                    if root then
                        safeFire(muscleEvent, "punch", "rightHand")
                        safeFire(muscleEvent, "punch", "leftHand")
                    end
                end
            end
            task.wait(0.02)
        end
    end)
end)

-- Auto Punch (No animation)
KillTab:AddSwitch("Auto Punch [No Animation - OP]", function(state)
    autoPunchNoAnim = state
    if state then
        task.spawn(function()
            while autoPunchNoAnim do
                local punch = LocalPlayer.Backpack:FindFirstChild("Punch") or LocalPlayer.Character:FindFirstChild("Punch")
                if punch then
                    if punch.Parent ~= LocalPlayer.Character then
                        punch.Parent = LocalPlayer.Character
                    end
                    if punch:FindFirstChild("attackTime") then
                        punch.attackTime.Value = 0
                    end
                    safeFire(muscleEvent, "punch", "rightHand")
                    safeFire(muscleEvent, "punch", "leftHand")
                end
                task.wait(0.008)
            end
        end)
    end
end)

-- Auto Equip Punch
KillTab:AddSwitch("Auto Equip Punch", function(state)
    autoEquipPunch = state
    if state then
        task.spawn(function()
            while autoEquipPunch do
                local punch = LocalPlayer.Backpack:FindFirstChild("Punch")
                if punch then punch.Parent = LocalPlayer.Character end
                task.wait(0.2)
            end
        end)
    end
end)

-- Remove Punch Animation
KillTab:AddButton("Remove Punch Animation", function()
    local blockedIds = {["rbxassetid://3638729053"] = true, ["rbxassetid://3638767427"] = true}
    local function block(char)
        local hum = char:FindFirstChild("Humanoid")
        if hum then
            for _, track in pairs(hum:GetPlayingAnimationTracks()) do
                if track.Animation and (blockedIds[track.Animation.AnimationId] or track.Name:lower():find("punch")) then
                    track:Stop()
                end
            end
            hum.AnimationPlayed:Connect(function(track)
                task.defer(function()
                    if track.Animation and (blockedIds[track.Animation.AnimationId] or track.Name:lower():find("punch")) then
                        track:Stop()
                    end
                end)
            end)
        end
    end
    if LocalPlayer.Character then block(LocalPlayer.Character) end
    LocalPlayer.CharacterAdded:Connect(block)
end)

-- Auto Slams Ground
KillTab:AddSwitch("Auto Ground Slams", function(state)
    autoGroundSlams = state
    if not state then return end
    task.spawn(function()
        while autoGroundSlams do
            local slam = LocalPlayer.Backpack:FindFirstChild("Ground Slam") or LocalPlayer.Character:FindFirstChild("Ground Slam")
            if slam then
                if slam.Parent == LocalPlayer.Backpack then
                    slam.Parent = LocalPlayer.Character
                end
                if slam:FindFirstChild("attackTime") then
                    slam.attackTime.Value = 0
                end
                safeFire(muscleEvent, "slam")
                slam:Activate()
            end
            task.wait(0.1)
        end
    end)
end)

-- Good Mode (Brawl)
KillTab:AddSwitch("Good Mode (Brawl Loop)", function(state)
    autoBrawl = state
    if not state then return end
    task.spawn(function()
        while autoBrawl do
            safeFire(ReplicatedStorage.rEvents.brawlEvent, "joinBrawl")
            task.wait(1)
        end
    end)
end)

-- Size Glitch NaN
KillTab:AddButton("Combo NaN (Size Glitch)", function()
    if ReplicatedStorage and ReplicatedStorage.rEvents and ReplicatedStorage.rEvents.changeSpeedSizeRemote then
        -- Trigger NaN size
        ReplicatedStorage.rEvents.changeSpeedSizeRemote:InvokeServer("changeSize", 0/0)
    end
end)

-- Follow / TP behind target
local followTargetName = nil
local followDropdown = KillTab:AddDropdown("Follow / TP Behind Player", function(displayName)
    followTargetName = displayName
    following = true
end)
for _, plr in ipairs(Players:GetPlayers()) do
    if plr ~= LocalPlayer then
        followDropdown:Add(plr.DisplayName)
    end
end)
KillTab:AddButton("Stop Following", function()
    following = false
    followTargetName = nil
end)

RunService.Heartbeat:Connect(function()
    if following and followTargetName then
        local target = Players:FindFirstChild(followTargetName)
        local myChar = LocalPlayer.Character
        local tChar = target and target.Character
        if myChar and tChar then
            local myRoot = myChar:FindFirstChild("HumanoidRootPart")
            local tRoot = tChar:FindFirstChild("HumanoidRootPart")
            if myRoot and tRoot then
                local pos = tRoot.Position - (tRoot.CFrame.LookVector * 4)
                myRoot.CFrame = CFrame.new(pos, tRoot.Position)
            end
        end
    end
end)

-- Change Time
local times = {"Morning", "Noon", "Afternoon", "Sunset", "Night", "Midnight", "Dawn", "Early Morning"}
local timeDropdown = KillTab:AddDropdown("Change Time", function(sel)
    Lighting.Brightness = 2
    Lighting.FogEnd = 100000
    if sel == "Morning" then
        Lighting.ClockTime = 6
        Lighting.Ambient = Color3.fromRGB(200, 200, 255)
    elseif sel == "Noon" then
        Lighting.ClockTime = 12
        Lighting.Brightness = 3
        Lighting.Ambient = Color3.fromRGB(255, 255, 255)
    elseif sel == "Afternoon" then
        Lighting.ClockTime = 16
        Lighting.Ambient = Color3.fromRGB(255, 220, 180)
    elseif sel == "Sunset" then
        Lighting.ClockTime = 18
        Lighting.Ambient = Color3.fromRGB(255, 150, 100)
        Lighting.FogEnd = 500
    elseif sel == "Night" then
        Lighting.ClockTime = 20
        Lighting.Brightness = 1.5
        Lighting.Ambient = Color3.fromRGB(100, 100, 150)
    elseif sel == "Midnight" then
        Lighting.ClockTime = 0
        Lighting.Brightness = 1
        Lighting.Ambient = Color3.fromRGB(50, 50, 100)
    elseif sel == "Dawn" then
        Lighting.ClockTime = 4
        Lighting.Ambient = Color3.fromRGB(180, 180, 220)
    elseif sel == "Early Morning" then
        Lighting.ClockTime = 2
        Lighting.Brightness = 1.2
        Lighting.Ambient = Color3.fromRGB(100, 120, 180)
    end
end)
for _, t in ipairs(times) do timeDropdown:Add(t) end

print("✅ Muscle Legends Kill Tab loaded successfully!")
