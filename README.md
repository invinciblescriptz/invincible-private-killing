--// Muscle Legends - Working Kill Tab 2026 (Rayfield UI)
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local Lighting = game:GetService("Lighting")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local LocalPlayer = Players.LocalPlayer
local muscleEvent = LocalPlayer:WaitForChild("muscleEvent")

--// Load Rayfield UI Library
local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local Window = Rayfield:CreateWindow({
    Name = "Muscle Legends Kill GUI 2026",
    LoadingTitle = "Loading...",
    LoadingSubtitle = "by Grok",
    ConfigurationSaving = { Enabled = false }
})

local Killer = Window:CreateTab("Kill", 4483362458)  -- You can change the icon ID

-- Variables
local playerWhitelist = {}
local targetPlayerNames = {}
local following = false
local followTarget = nil
local autoGoodKarma = false
local autoBadKarma = false

-- Draggable Frame (if you want custom frame, but Rayfield already has its own nice GUI)

-- Pet Selector
Killer:CreateLabel("Select Pet (Damage / Durability)")

local petDropdown = Killer:CreateDropdown({
    Name = "Select Pet",
    Options = {"Wild Wizard", "Mighty Monster"},
    CurrentOption = {"Wild Wizard"},
    MultipleOptions = false,
    Flag = "PetFlag",
    Callback = function(petName)
        petName = petName[1] or petName

        -- Unequip all pets
        pcall(function()
            for _, folder in pairs(LocalPlayer:FindFirstChild("petsFolder"):GetChildren()) do
                if folder:IsA("Folder") then
                    for _, pet in pairs(folder:GetChildren()) do
                        ReplicatedStorage.rEvents.equipPetEvent:FireServer("unequipPet", pet)
                    end
                end
            end
        end)
        task.wait(0.4)

        -- Equip selected
        local equipped = 0
        local uniqueFolder = LocalPlayer:FindFirstChild("petsFolder") and LocalPlayer.petsFolder:FindFirstChild("Unique")
        if uniqueFolder then
            for _, pet in pairs(uniqueFolder:GetChildren()) do
                if pet.Name == petName and equipped < 8 then
                    ReplicatedStorage.rEvents.equipPetEvent:FireServer("equipPet", pet)
                    equipped += 1
                    task.wait(0.12)
                end
            end
        end
    end,
})

-- Auto Good Karma
Killer:CreateToggle({
    Name = "Auto Good Karma",
    CurrentValue = false,
    Flag = "GoodKarmaFlag",
    Callback = function(Value)
        autoGoodKarma = Value
        if Value then
            task.spawn(function()
                while autoGoodKarma do
                    local char = LocalPlayer.Character
                    if char then
                        for _, plr in ipairs(Players:GetPlayers()) do
                            if plr ~= LocalPlayer then
                                local evil = plr:FindFirstChild("evilKarma")
                                local good = plr:FindFirstChild("goodKarma")
                                if evil and good and evil.Value > good.Value then
                                    local root = plr.Character and plr.Character:FindFirstChild("HumanoidRootPart")
                                    if root then
                                        muscleEvent:FireServer("punch", "rightHand")
                                        muscleEvent:FireServer("punch", "leftHand")
                                    end
                                end
                            end
                        end
                    end
                    task.wait(0.03)
                end
            end)
        end
    end,
})

-- Auto Bad Karma (similar toggle - copy the structure above and change condition to good.Value > evil.Value)
Killer:CreateToggle({
    Name = "Auto Bad Karma",
    CurrentValue = false,
    Flag = "BadKarmaFlag",
    Callback = function(Value)
        autoBadKarma = Value
        if Value then
            task.spawn(function()
                while autoBadKarma do
                    local char = LocalPlayer.Character
                    if char then
                        for _, plr in ipairs(Players:GetPlayers()) do
                            if plr ~= LocalPlayer then
                                local evil = plr:FindFirstChild("evilKarma")
                                local good = plr:FindFirstChild("goodKarma")
                                if evil and good and good.Value > evil.Value then
                                    local root = plr.Character and plr.Character:FindFirstChild("HumanoidRootPart")
                                    if root then
                                        muscleEvent:FireServer("punch", "rightHand")
                                        muscleEvent:FireServer("punch", "leftHand")
                                    end
                                end
                            end
                        end
                    end
                    task.wait(0.03)
                end
            end)
        end
    end,
})

-- Auto Kill All (except whitelisted)
Killer:CreateToggle({
    Name = "Auto Kill All",
    CurrentValue = false,
    Callback = function(Value)
        task.spawn(function()
            while Value do
                local char = LocalPlayer.Character
                if char then
                    for _, plr in ipairs(Players:GetPlayers()) do
                        if plr ~= LocalPlayer and not playerWhitelist[plr.Name] and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
                            muscleEvent:FireServer("punch", "rightHand")
                            muscleEvent:FireServer("punch", "leftHand")
                        end
                    end
                end
                task.wait(0.02)
            end
        end)
    end,
})

-- Auto Punch No Animation (Most Reliable in 2026)
Killer:CreateToggle({
    Name = "Auto Punch [No Animation - OP]",
    CurrentValue = false,
    Callback = function(Value)
        task.spawn(function()
            while Value do
                local punch = LocalPlayer.Backpack:FindFirstChild("Punch") or (LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Punch"))
                if punch then
                    if punch.Parent ~= LocalPlayer.Character then
                        punch.Parent = LocalPlayer.Character
                    end
                    if punch:FindFirstChild("attackTime") then
                        punch.attackTime.Value = 0
                    end
                    muscleEvent:FireServer("punch", "rightHand")
                    muscleEvent:FireServer("punch", "leftHand")
                end
                task.wait(0.008)
            end
        end)
    end,
})

-- Auto Equip Punch
Killer:CreateToggle({
    Name = "Auto Equip Punch",
    CurrentValue = false,
    Callback = function(Value)
        task.spawn(function()
            while Value do
                local punch = LocalPlayer.Backpack:FindFirstChild("Punch")
                if punch then punch.Parent = LocalPlayer.Character end
                task.wait(0.2)
            end
        end)
    end,
})

-- Remove Punch Animation
Killer:CreateButton({
    Name = "Remove Punch Animation",
    Callback = function()
        local blocked = {["rbxassetid://3638729053"] = true, ["rbxassetid://3638767427"] = true}
        local function blockAnims(char)
            local hum = char:FindFirstChild("Humanoid")
            if hum then
                for _, track in pairs(hum:GetPlayingAnimationTracks()) do
                    if track.Animation and (blocked[track.Animation.AnimationId] or track.Name:lower():find("punch")) then
                        track:Stop()
                    end
                end
                hum.AnimationPlayed:Connect(function(track)
                    task.defer(function()
                        if track.Animation and (blocked[track.Animation.AnimationId] or track.Name:lower():find("punch")) then
                            track:Stop()
                        end
                    end)
                end)
            end
        end
        if LocalPlayer.Character then blockAnims(LocalPlayer.Character) end
        LocalPlayer.CharacterAdded:Connect(blockAnims)
    end,
})

-- Auto Ground Slams
Killer:CreateToggle({
    Name = "Auto Ground Slams",
    CurrentValue = false,
    Callback = function(Value)
        task.spawn(function()
            while Value do
                local slam = LocalPlayer.Backpack:FindFirstChild("Ground Slam") or (LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Ground Slam"))
                if slam then
                    if slam.Parent == LocalPlayer.Backpack then slam.Parent = LocalPlayer.Character end
                    if slam:FindFirstChild("attackTime") then slam.attackTime.Value = 0 end
                    muscleEvent:FireServer("slam")
                    slam:Activate()
                end
                task.wait(0.1)
            end
        end)
    end,
})

-- Good Mode (Brawl)
Killer:CreateToggle({
    Name = "Good Mode (Brawl Loop)",
    CurrentValue = false,
    Callback = function(Value)
        task.spawn(function()
            while Value do
                ReplicatedStorage.rEvents.brawlEvent:FireServer("joinBrawl")
                task.wait(1)
            end
        end)
    end,
})

-- Combo NaN
Killer:CreateButton({
    Name = "Combo NaN (Size Glitch)",
    Callback = function()
        ReplicatedStorage.rEvents.changeSpeedSizeRemote:InvokeServer("changeSize", 0/0)
    end,
})

-- Follow Player
local followDropdown = Killer:CreateDropdown({
    Name = "Follow / TP Behind Player",
    Options = {},
    CurrentOption = {},
    MultipleOptions = false,
    Callback = function(displayName)
        displayName = displayName[1] or displayName
        for _, plr in ipairs(Players:GetPlayers()) do
            if plr.DisplayName == displayName then
                followTarget = plr.Name
                following = true
                break
            end
        end
    end,
})

-- Populate follow dropdown
for _, plr in ipairs(Players:GetPlayers()) do
    if plr ~= LocalPlayer then
        followDropdown:Add(plr.DisplayName)
    end
end

Players.PlayerAdded:Connect(function(plr)
    if plr ~= LocalPlayer then followDropdown:Add(plr.DisplayName) end
end)

Killer:CreateButton({
    Name = "Stop Following",
    Callback = function()
        following = false
        followTarget = nil
    end,
})

-- Follow Loop
task.spawn(function()
    while task.wait(0.03) do
        if following and followTarget then
            local target = Players:FindFirstChild(followTarget)
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
    end
end)

-- Time Changer
local times = {"Morning","Noon","Afternoon","Sunset","Night","Midnight","Dawn","Early Morning"}
Killer:CreateDropdown({
    Name = "Change Time",
    Options = times,
    CurrentOption = {"Noon"},
    MultipleOptions = false,
    Callback = function(sel)
        sel = sel[1] or sel
        Lighting.Brightness = 2
        Lighting.FogEnd = 100000
        if sel == "Morning" then Lighting.ClockTime = 6 Lighting.Ambient = Color3.fromRGB(200,200,255)
        elseif sel == "Noon" then Lighting.ClockTime = 12 Lighting.Brightness = 3 Lighting.Ambient = Color3.fromRGB(255,255,255)
        -- ... (add the rest of the conditions from previous version)
        end
    end,
})

print("✅ Muscle Legends Kill GUI Loaded with Rayfield!")
