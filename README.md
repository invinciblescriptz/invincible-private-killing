-- // Muscle Legends OP Kill Tab - FIXED 2026 \\ --
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Lighting = game:GetService("Lighting")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local LocalPlayer = Players.LocalPlayer

-- Wait for character to load
if not LocalPlayer.Character then
    LocalPlayer.CharacterAdded:Wait()
end

-- Find muscleEvent (most common locations)
local muscleEvent = ReplicatedStorage:FindFirstChild("muscleEvent") 
    or LocalPlayer:FindFirstChild("muscleEvent") 
    or ReplicatedStorage.rEvents:FindFirstChild("muscleEvent")

if not muscleEvent then
    warn("❌ muscleEvent not found! Script may not work properly.")
end

-- Load Kavo UI Library (fixed & most reliable link)
local Kavo = loadstring(game:HttpGet("https://raw.githubusercontent.com/xHeptc/Kavo-UI-Library/main/source.lua"))()
local Window = Kavo.new({
    Title = "Muscle Legends OP",
    Center = true,
    AutoShow = true  -- This forces it to show immediately
})

local KillTab = Window:CreateTab("Kill")

-- Flags
local autoGoodKarma = false
local autoBadKarma = false
local killAllToggle = false
local targetKillActive = false
local autoPunchNoAnim = false
local autoEquipPunch = false
local autoGroundSlams = false
local autoBrawl = false
local following = false
local followTargetName = nil

local playerWhitelist = {}
local targetPlayerNames = {}

local function safeFire(event, ...)
    if event then
        pcall(function()
            event:FireServer(...)
        end)
    end
end

local function getCharacterParts(plr)
    if plr and plr.Character then
        return plr.Character:FindFirstChild("HumanoidRootPart"), plr.Character:FindFirstChild("Humanoid")
    end
    return nil, nil
end

-- ==================== PETS ====================
local PetsSection = KillTab:AddSection("Pets")

PetsSection:AddLabel("Select Pet (Damage / Durability)")

local petDropdown = PetsSection:AddDropdown("Select Pet", function(petName)
    -- Unequip all first
    pcall(function()
        for _, folder in pairs(LocalPlayer:FindFirstChild("petsFolder"):GetChildren()) do
            if folder:IsA("Folder") then
                for _, pet in pairs(folder:GetChildren()) do
                    ReplicatedStorage.rEvents.equipPetEvent:FireServer("unequipPet", pet)
                end
            end
        end
    end)
    task.wait(0.5)

    local equipped = 0
    local uniqueFolder = LocalPlayer.petsFolder:FindFirstChild("Unique")
    if uniqueFolder then
        for _, pet in pairs(uniqueFolder:GetChildren()) do
            if pet.Name == petName and equipped < 8 then
                ReplicatedStorage.rEvents.equipPetEvent:FireServer("equipPet", pet)
                equipped += 1
                task.wait(0.15)
            end
        end
    end
end)

petDropdown:Add("Wild Wizard")
petDropdown:Add("Mighty Monster")
-- Add more pets here if you want

-- ==================== KARMA AUTO ATTACK ====================
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
                            local shouldAttack = isGood and (evil.Value > good.Value) or (good.Value > evil.Value)
                            if shouldAttack then
                                local root = plr.Character and plr.Character:FindFirstChild("HumanoidRootPart")
                                if root then
                                    local rh = char:FindFirstChild("RightHand")
                                    local lh = char:FindFirstChild("LeftHand")
                                    if rh and lh then
                                        firetouchinterest(rh, root, 0)
                                        firetouchinterest(lh, root, 0)
                                    end
                                    safeFire(muscleEvent, "punch", "rightHand")
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

-- ==================== WHITELIST ====================
local WhitelistSection = KillTab:AddSection("Whitelist")

WhitelistSection:AddSwitch("Auto Whitelist Friends", function(state)
    playerWhitelist = {}
    if state then
        for _, plr in ipairs(Players:GetPlayers()) do
            if plr ~= LocalPlayer and LocalPlayer:IsFriendsWith(plr.UserId) then
                playerWhitelist[plr.Name] = true
            end
        end
    end
end)

WhitelistSection:AddTextBox("Whitelist Player (Name)", function(txt)
    if Players:FindFirstChild(txt) then
        playerWhitelist[txt] = true
    end
end)

WhitelistSection:AddTextBox("Unwhitelist Player (Name)", function(txt)
    playerWhitelist[txt] = nil
end)

-- ==================== AUTO KILL ALL ====================
KillTab:AddSwitch("Auto Kill All (Skip Whitelist)", function(state)
    killAllToggle = state
    if not state then return end

    task.spawn(function()
        while killAllToggle do
            local char = LocalPlayer.Character
            if char then
                for _, plr in ipairs(Players:GetPlayers()) do
                    if plr ~= LocalPlayer and not playerWhitelist[plr.Name] then
                        safeFire(muscleEvent, "punch", "rightHand")
                        safeFire(muscleEvent, "punch", "leftHand")
                    end
                end
            end
            task.wait(0.02)
        end
    end)
end)

-- ==================== TARGET KILL ====================
local TargetSection = KillTab:AddSection("Target Kill")

local targetDropdown = TargetSection:AddDropdown("Add Target (DisplayName)", function(displayName)
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr.DisplayName == displayName and not table.find(targetPlayerNames, plr.Name) then
            table.insert(targetPlayerNames, plr.Name)
            break
        end
    end
end)

-- Populate dropdown
for _, plr in ipairs(Players:GetPlayers()) do
    if plr ~= LocalPlayer then
        targetDropdown:Add(plr.DisplayName)
    end
end

Players.PlayerAdded:Connect(function(plr)
    if plr ~= LocalPlayer then
        targetDropdown:Add(plr.DisplayName)
    end
end)

TargetSection:AddButton("Clear All Targets", function()
    targetPlayerNames = {}
end)

KillTab:AddSwitch("Kill Selected Targets", function(state)
    targetKillActive = state
    if not state then return end

    task.spawn(function()
        while targetKillActive do
            for _, name in ipairs(targetPlayerNames) do
                local plr = Players:FindFirstChild(name)
                if plr and plr.Character then
                    safeFire(muscleEvent, "punch", "rightHand")
                    safeFire(muscleEvent, "punch", "leftHand")
                end
            end
            task.wait(0.02)
        end
    end)
end)

-- ==================== AUTO PUNCH (No Animation) ====================
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

KillTab:AddSwitch("Auto Equip Punch", function(state)
    autoEquipPunch = state
    if state then
        task.spawn(function()
            while autoEquipPunch do
                local punch = LocalPlayer.Backpack:FindFirstChild("Punch")
                if punch then
                    punch.Parent = LocalPlayer.Character
                end
                task.wait(0.2)
            end
        end)
    end
end)

-- ==================== GROUND SLAM ====================
KillTab:AddSwitch("Auto Ground Slams", function(state)
    autoGroundSlams = state
    if state then
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
    end
end)

-- ==================== BRAWL / GOOD MODE ====================
KillTab:AddSwitch("Good Mode (Auto Join Brawl)", function(state)
    autoBrawl = state
    if state then
        task.spawn(function()
            while autoBrawl do
                safeFire(ReplicatedStorage.rEvents.brawlEvent, "joinBrawl")
                task.wait(1)
            end
        end)
    end
end)

-- ==================== SIZE GLITCH ====================
KillTab:AddButton("Combo NaN Size Glitch", function()
    local sizeRemote = ReplicatedStorage.rEvents:FindFirstChild("changeSpeedSizeRemote")
    if sizeRemote then
        sizeRemote:InvokeServer("changeSize", 0/0)
    else
        warn("changeSpeedSizeRemote not found")
    end
end)

-- ==================== FOLLOW / TP BEHIND ====================
local followDropdown = KillTab:AddDropdown("Follow / TP Behind Player", function(displayName)
    followTargetName = displayName
    following = true
end)

for _, plr in ipairs(Players:GetPlayers()) do
    if plr ~= LocalPlayer then
        followDropdown:Add(plr.DisplayName)
    end
end

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

-- ==================== CHANGE TIME ====================
local times = {"Morning","Noon","Afternoon","Sunset","Night","Midnight","Dawn","Early Morning"}
local timeDropdown = KillTab:AddDropdown("Change Time", function(sel)
    Lighting.Brightness = 2
    Lighting.FogEnd = 100000

    if sel == "Morning" then
        Lighting.ClockTime = 6
        Lighting.Ambient = Color3.fromRGB(200,200,255)
    elseif sel == "Noon" then
        Lighting.ClockTime = 12
        Lighting.Brightness = 3
        Lighting.Ambient = Color3.fromRGB(255,255,255)
    elseif sel == "Afternoon" then
        Lighting.ClockTime = 16
        Lighting.Ambient = Color3.fromRGB(255,220,180)
    elseif sel == "Sunset" then
        Lighting.ClockTime = 18
        Lighting.Ambient = Color3.fromRGB(255,150,100)
        Lighting.FogEnd = 500
    elseif sel == "Night" then
        Lighting.ClockTime = 20
        Lighting.Brightness = 1.5
        Lighting.Ambient = Color3.fromRGB(100,100,150)
    elseif sel == "Midnight" then
        Lighting.ClockTime = 0
        Lighting.Brightness = 1
        Lighting.Ambient = Color3.fromRGB(50,50,100)
    elseif sel == "Dawn" then
        Lighting.ClockTime = 4
        Lighting.Ambient = Color3.fromRGB(180,180,220)
    elseif sel == "Early Morning" then
        Lighting.ClockTime = 2
        Lighting.Brightness = 1.2
        Lighting.Ambient = Color3.fromRGB(100,120,180)
    end
end)

for _, t in ipairs(times) do
    timeDropdown:Add(t)
end

print("✅ Muscle Legends OP Script - Fully Fixed & Loaded!")
