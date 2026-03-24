-- // Muscle Legends OP - Kavo UI Fixed (No Rayfield) - Full Script \\ 
-- Paste this entire chunk at once

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Lighting = game:GetService("Lighting")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local LocalPlayer = Players.LocalPlayer

-- Wait for game to load
task.wait(1)
if not LocalPlayer.Character then
    LocalPlayer.CharacterAdded:Wait()
end
task.wait(0.5)

-- Find muscleEvent
local muscleEvent = ReplicatedStorage:FindFirstChild("muscleEvent") 
    or LocalPlayer:FindFirstChild("muscleEvent") 
    or ReplicatedStorage.rEvents:FindFirstChild("muscleEvent")

if not muscleEvent then
    warn("⚠️ muscleEvent not found! Some kill features may not work.")
end

-- Load Kavo UI (most reliable method in 2026)
local Library = nil
pcall(function()
    Library = loadstring(game:HttpGet("https://raw.githubusercontent.com/xHeptc/Kavo-UI-Library/main/source.lua"))()
end)

if not Library then
    -- Backup loadstring (often works better with some executors)
    pcall(function()
        Library = loadstring(game:HttpGet("https://raw.githubusercontent.com/bloodball/-back-ups-for-libs/main/kav"))()
    end)
end

if not Library then
    error("❌ Failed to load Kavo UI. Try rejoining or a different executor.")
end

-- Create Window
local Window = Library:CreateLib("Muscle Legends OP", "DarkTheme")  -- Try "BloodTheme" or "Synapse" if you want different look

local KillTab = Window:NewTab("Kill")

-- Flags
local autoGoodKarma = false
local autoBadKarma = false
local killAllToggle = false
local targetKillActive = false
local autoPunchNoAnim = false
local autoGroundSlams = false
local autoBrawl = false
local following = false
local followTargetName = nil

local playerWhitelist = {}
local targetPlayerNames = {}

local function safeFire(event, ...)
    if event then pcall(function() event:FireServer(...) end) end
end

local function getRoot(plr)
    return plr and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart")
end

-- ====================== PETS ======================
KillTab:NewSection("Pets")

local petDropdown = KillTab:NewDropdown("Select Pet", "Choose pet to equip", {"Wild Wizard", "Mighty Monster"}, function(petName)
    -- Unequip all
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

-- ====================== KARMA ======================
local function karmaLoop(isGood)
    task.spawn(function()
        while (isGood and autoGoodKarma) or (not isGood and autoBadKarma) do
            if LocalPlayer.Character then
                for _, plr in ipairs(Players:GetPlayers()) do
                    if plr ~= LocalPlayer then
                        local evil = plr:FindFirstChild("evilKarma")
                        local good = plr:FindFirstChild("goodKarma")
                        if evil and good then
                            local condition = isGood and (evil.Value > good.Value) or (good.Value > evil.Value)
                            if condition then
                                safeFire(muscleEvent, "punch", "rightHand")
                            end
                        end
                    end
                end
            end
            task.wait(0.03)
        end
    end)
end

KillTab:NewToggle("Auto Good Karma", "Attacks high evil karma players", false, function(state)
    autoGoodKarma = state
    if state then karmaLoop(true) end
end)

KillTab:NewToggle("Auto Bad Karma", "Attacks high good karma players", false, function(state)
    autoBadKarma = state
    if state then karmaLoop(false) end
end)

-- ====================== AUTO KILL ALL ======================
KillTab:NewToggle("Auto Kill All (Skip Whitelist)", "Kills everyone not whitelisted", false, function(state)
    killAllToggle = state
    if not state then return end
    task.spawn(function()
        while killAllToggle do
            for _, plr in ipairs(Players:GetPlayers()) do
                if plr ~= LocalPlayer and not playerWhitelist[plr.Name] then
                    safeFire(muscleEvent, "punch", "rightHand")
                    safeFire(muscleEvent, "punch", "leftHand")
                end
            end
            task.wait(0.02)
        end
    end)
end)

-- ====================== TARGET KILL ======================
KillTab:NewSection("Target Kill")

local targetDropdown = KillTab:NewDropdown("Add Target", "Select player by display name", {}, function(displayName)
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr.DisplayName == displayName and not table.find(targetPlayerNames, plr.Name) then
            table.insert(targetPlayerNames, plr.Name)
            break
        end
    end
end)

-- Auto populate
for _, plr in ipairs(Players:GetPlayers()) do
    if plr ~= LocalPlayer then
        targetDropdown:Add(plr.DisplayName)
    end
end

Players.PlayerAdded:Connect(function(plr)
    if plr ~= LocalPlayer then targetDropdown:Add(plr.DisplayName) end
end)

KillTab:NewButton("Clear All Targets", "Remove all selected targets", function()
    targetPlayerNames = {}
end)

KillTab:NewToggle("Kill Selected Targets", "", false, function(state)
    targetKillActive = state
    if not state then return end
    task.spawn(function()
        while targetKillActive do
            for _, name in ipairs(targetPlayerNames) do
                local plr = Players:FindFirstChild(name)
                if plr then
                    safeFire(muscleEvent, "punch", "rightHand")
                    safeFire(muscleEvent, "punch", "leftHand")
                end
            end
            task.wait(0.02)
        end
    end)
end)

-- ====================== AUTO PUNCH & SLAMS ======================
KillTab:NewToggle("Auto Punch [No Animation - OP]", "Super fast punching", false, function(state)
    autoPunchNoAnim = state
    if state then
        task.spawn(function()
            while autoPunchNoAnim do
                local punch = LocalPlayer.Backpack:FindFirstChild("Punch") or LocalPlayer.Character:FindFirstChild("Punch")
                if punch then
                    if punch.Parent ~= LocalPlayer.Character then punch.Parent = LocalPlayer.Character end
                    if punch:FindFirstChild("attackTime") then punch.attackTime.Value = 0 end
                    safeFire(muscleEvent, "punch", "rightHand")
                    safeFire(muscleEvent, "punch", "leftHand")
                end
                task.wait(0.008)
            end
        end)
    end
end)

KillTab:NewToggle("Auto Ground Slams", "", false, function(state)
    autoGroundSlams = state
    if state then
        task.spawn(function()
            while autoGroundSlams do
                local slam = LocalPlayer.Backpack:FindFirstChild("Ground Slam") or LocalPlayer.Character:FindFirstChild("Ground Slam")
                if slam then
                    if slam.Parent == LocalPlayer.Backpack then slam.Parent = LocalPlayer.Character end
                    if slam:FindFirstChild("attackTime") then slam.attackTime.Value = 0 end
                    safeFire(muscleEvent, "slam")
                    slam:Activate()
                end
                task.wait(0.1)
            end
        end)
    end
end)

-- ====================== EXTRAS ======================
KillTab:NewSection("Extras")

KillTab:NewButton("NaN Size Glitch", "Makes character huge/glitched", function()
    local remote = ReplicatedStorage.rEvents:FindFirstChild("changeSpeedSizeRemote")
    if remote then
        remote:InvokeServer("changeSize", 0/0)
    end
end)

KillTab:NewToggle("Auto Join Brawl (Good Mode)", "", false, function(state)
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

-- Follow Player
local followDropdown = KillTab:NewDropdown("Follow / TP Behind Player", "", {}, function(displayName)
    followTargetName = displayName
    following = true
end)

for _, plr in ipairs(Players:GetPlayers()) do
    if plr ~= LocalPlayer then
        followDropdown:Add(plr.DisplayName)
    end
end

KillTab:NewButton("Stop Following", "", function()
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
                local pos = tRoot.Position - (tRoot.CFrame.LookVector * 5)
                myRoot.CFrame = CFrame.new(pos, tRoot.Position)
            end
        end
    end
end)

-- Time Changer
KillTab:NewSection("World Time")

local times = {"Morning","Noon","Afternoon","Sunset","Night","Midnight","Dawn","Early Morning"}

local timeDropdown = KillTab:NewDropdown("Change Time", "", times, function(sel)
    Lighting.Brightness = 2
    Lighting.FogEnd = 100000

    if sel == "Morning" then
        Lighting.ClockTime = 6; Lighting.Ambient = Color3.fromRGB(200,200,255)
    elseif sel == "Noon" then
        Lighting.ClockTime = 12; Lighting.Brightness = 3; Lighting.Ambient = Color3.fromRGB(255,255,255)
    elseif sel == "Afternoon" then
        Lighting.ClockTime = 16; Lighting.Ambient = Color3.fromRGB(255,220,180)
    elseif sel == "Sunset" then
        Lighting.ClockTime = 18; Lighting.Ambient = Color3.fromRGB(255,150,100); Lighting.FogEnd = 500
    elseif sel == "Night" then
        Lighting.ClockTime = 20; Lighting.Brightness = 1.5; Lighting.Ambient = Color3.fromRGB(100,100,150)
    elseif sel == "Midnight" then
        Lighting.ClockTime = 0; Lighting.Brightness = 1; Lighting.Ambient = Color3.fromRGB(50,50,100)
    elseif sel == "Dawn" then
        Lighting.ClockTime = 4; Lighting.Ambient = Color3.fromRGB(180,180,220)
    elseif sel == "Early Morning" then
        Lighting.ClockTime = 2; Lighting.Brightness = 1.2; Lighting.Ambient = Color3.fromRGB(100,120,180)
    end
end)

print("✅ Muscle Legends OP - Kavo UI Loaded! GUI should appear now.")

-- Notification if library supports it
pcall(function()
    Library:Notify("Success!", "Script loaded correctly. Enjoy the kills!", 5)
end)
