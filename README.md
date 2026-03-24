-- // Muscle Legends OP - FIXED GUI (March 2026) \\ --

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Lighting = game:GetService("Lighting")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local LocalPlayer = Players.LocalPlayer

-- Wait for everything to load properly
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
    warn("muscleEvent not found!")
end

-- ==================== LOAD KAVO UI (More Reliable Method) ====================
local Library = nil
pcall(function()
    Library = loadstring(game:HttpGet("https://raw.githubusercontent.com/xHeptc/Kavo-UI-Library/main/source.lua"))()
end)

if not Library then
    -- Backup loadstring (often more stable)
    Library = loadstring(game:HttpGet("https://raw.githubusercontent.com/bloodball/-back-ups-for-libs/main/kav"))()
end

if not Library then
    error("Failed to load Kavo UI Library! Try a different executor or rejoin.")
end

-- Create Window (Correct Kavo syntax)
local Window = Library:CreateLib("Muscle Legends OP", "DarkTheme")  -- or "BloodTheme", "Synapse", etc.

local KillTab = Window:NewTab("Kill")

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
    if event then pcall(function() event:FireServer(...) end) end
end

local function getRoot(plr)
    if plr and plr.Character then
        return plr.Character:FindFirstChild("HumanoidRootPart")
    end
    return nil
end

-- ==================== PETS ====================
local PetsSection = KillTab:NewSection("Pets")

PetsSection:NewLabel("Select Pet")

local petDropdown = PetsSection:NewDropdown("Select Pet", "Choose a pet", {"Wild Wizard", "Mighty Monster"}, function(petName)
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
    local unique = LocalPlayer.petsFolder:FindFirstChild("Unique")
    if unique then
        for _, pet in pairs(unique:GetChildren()) do
            if pet.Name == petName and equipped < 8 then
                ReplicatedStorage.rEvents.equipPetEvent:FireServer("equipPet", pet)
                equipped += 1
                task.wait(0.15)
            end
        end
    end
end)

-- ==================== KARMA ====================
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
                            local attack = isGood and (evil.Value > good.Value) or (good.Value > evil.Value)
                            if attack then
                                local root = getRoot(plr)
                                if root then
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

KillTab:NewToggle("Auto Good Karma", "Attacks players with more evil karma", false, function(state)
    autoGoodKarma = state
    if state then karmaLoop(true) end
end)

KillTab:NewToggle("Auto Bad Karma", "Attacks players with more good karma", false, function(state)
    autoBadKarma = state
    if state then karmaLoop(false) end
end)

-- ==================== AUTO KILL ALL ====================
KillTab:NewToggle("Auto Kill All", "Kills everyone except whitelisted", false, function(state)
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

-- ==================== TARGET KILL ====================
local TargetSection = KillTab:NewSection("Target Kill")

local targetDropdown = TargetSection:NewDropdown("Add Target", "Select player", {}, function(displayName)
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr.DisplayName == displayName and not table.find(targetPlayerNames, plr.Name) then
            table.insert(targetPlayerNames, plr.Name)
            break
        end
    end
end)

-- Auto populate players
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

TargetSection:NewButton("Clear Targets", "Remove all targets", function()
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

-- ==================== AUTO PUNCH & SLAMS ====================
KillTab:NewToggle("Auto Punch [No Animation]", "Very fast punching", false, function(state)
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

-- ==================== OTHER FEATURES (Size Glitch, Follow, Time) ====================
KillTab:NewButton("NaN Size Glitch", "Makes you huge/tiny", function()
    local remote = ReplicatedStorage.rEvents:FindFirstChild("changeSpeedSizeRemote")
    if remote then
        remote:InvokeServer("changeSize", 0/0)
    end
end)

-- Follow player (simple version)
local followDropdown = KillTab:NewDropdown("Follow Player", "", {}, function(displayName)
    followTargetName = displayName
    following = true
end)

for _, plr in ipairs(Players:GetPlayers()) do
    if plr ~= LocalPlayer then followDropdown:Add(plr.DisplayName) end
end

KillTab:NewButton("Stop Following", "", function()
    following = false
end)

RunService.Heartbeat:Connect(function()
    if following and followTargetName then
        local target = Players:FindFirstChild(followTargetName)
        if target and target.Character and LocalPlayer.Character then
            local myRoot = LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
            local tRoot = target.Character:FindFirstChild("HumanoidRootPart")
            if myRoot and tRoot then
                local pos = tRoot.Position - tRoot.CFrame.LookVector * 5
                myRoot.CFrame = CFrame.new(pos, tRoot.Position)
            end
        end
    end
end)

print("✅ Muscle Legends OP Script Loaded! GUI should now appear.")

-- If the GUI still doesn't show, try these:
-- 1. Rejoin the game
-- 2. Use a different executor (try Solara, Wave, or Fluxus)
-- 3. Execute this single line first to test Kavo:
--    loadstring(game:HttpGet("https://raw.githubusercontent.com/xHeptc/Kavo-UI-Library/main/source.lua"))():CreateLib("Test", "DarkTheme")
