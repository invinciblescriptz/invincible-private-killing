--// Muscle Legends OP Kill Tab - Fixed & Optimized 2026
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local Lighting = game:GetService("Lighting")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RunService = game:GetService("RunService")

local LocalPlayer = Players.LocalPlayer
local muscleEvent = LocalPlayer:WaitForChild("muscleEvent")

--// GUI Setup (replace "window" with your actual UI library)
local gui = Instance.new("ScreenGui")
gui.Name = "MuscleKillUI"
gui.ResetOnSpawn = false
gui.Parent = LocalPlayer:WaitForChild("PlayerGui")

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 520, 0, 600)
frame.Position = UDim2.new(0.5, -260, 0.5, -300)
frame.BackgroundColor3 = Color3.fromRGB(0, 10, 0)
frame.BorderSizePixel = 0

local function makeDraggable(f)
    local dragging, dragStart, startPos
    f.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            dragStart = input.Position
            startPos = f.Position
        end
    end)
    UserInputService.InputChanged:Connect(function(input)
        if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
            local delta = input.Position - dragStart
            f.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        end
    end)
    f.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end
    end)
end
makeDraggable(frame)

-- Assuming you have a UI library like Linoria / Rayfield / etc.
local window = --[[ PUT YOUR WINDOW HERE ]] 
local Killer = window:AddTab("Kill")

-- Variables
local playerWhitelist = {}
local targetPlayerNames = {}
local following = false
local followTarget = nil
local targetPlayerName = nil

--// Pet Selector
Killer:AddLabel("Select Pet (Damage / Durability)", {TextSize = 18, Font = Enum.Font.Merriweather})

local petDropdown = Killer:AddDropdown("Select Pet", function(petName)
    -- Unequip all
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

    -- Equip up to 8
    local equipped = 0
    for _, pet in pairs(LocalPlayer.petsFolder.Unique:GetChildren()) do
        if pet.Name == petName and equipped < 8 then
            ReplicatedStorage.rEvents.equipPetEvent:FireServer("equipPet", pet)
            equipped += 1
            task.wait(0.12)
        end
    end
end)

petDropdown:Add("Wild Wizard")
petDropdown:Add("Mighty Monster")

--// Auto Good / Bad Karma (using touch + event backup)
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
                                local root = plr.Character and plr.Character:FindFirstChild("HumanoidRootPart")
                                if root then
                                    -- Try touch + direct event
                                    local rh = char:FindFirstChild("RightHand")
                                    local lh = char:FindFirstChild("LeftHand")
                                    if rh and lh then
                                        firetouchinterest(rh, root, 0)
                                        firetouchinterest(lh, root, 0)
                                    end
                                    muscleEvent:FireServer("punch", "rightHand")
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

local autoGoodKarma, autoBadKarma = false, false

Killer:AddSwitch("Auto Good Karma", function(state)
    autoGoodKarma = state
    if state then karmaLoop(true) end
end)

Killer:AddSwitch("Auto Bad Karma", function(state)
    autoBadKarma = state
    if state then karmaLoop(false) end
end)

--// Whitelist
Killer:AddSwitch("Auto Whitelist Friends", function(state)
    if state then
        for _, plr in ipairs(Players:GetPlayers()) do
            if plr ~= LocalPlayer and LocalPlayer:IsFriendsWith(plr.UserId) then
                playerWhitelist[plr.Name] = true
            end
        end
    else
        playerWhitelist = {}
    end
end)

Killer:AddTextBox("Whitelist Player", function(txt)
    local plr = Players:FindFirstChild(txt)
    if plr then playerWhitelist[plr.Name] = true end
end)

Killer:AddTextBox("Unwhitelist Player", function(txt)
    playerWhitelist[txt] = nil
end)

--// Auto Kill All (except whitelisted)
Killer:AddSwitch("Auto Kill All", function(state)
    task.spawn(function()
        while state do
            local char = LocalPlayer.Character
            if char then
                for _, plr in ipairs(Players:GetPlayers()) do
                    if plr ~= LocalPlayer and not playerWhitelist[plr.Name] and plr.Character then
                        local root = plr.Character:FindFirstChild("HumanoidRootPart")
                        if root then
                            muscleEvent:FireServer("punch", "rightHand")
                            muscleEvent:FireServer("punch", "leftHand")
                        end
                    end
                end
            end
            task.wait(0.02)
        end
    end)
end)

--// Target Kill
local targetDropdown = Killer:AddDropdown("Add Kill Target", function(displayName)
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr.DisplayName == displayName and not table.find(targetPlayerNames, plr.Name) then
            table.insert(targetPlayerNames, plr.Name)
            break
        end
    end
end)

Killer:AddButton("Clear All Targets", function()
    targetPlayerNames = {}
end)

-- Populate dropdown
for _, plr in ipairs(Players:GetPlayers()) do
    if plr ~= LocalPlayer then targetDropdown:Add(plr.DisplayName) end
end

Players.PlayerAdded:Connect(function(plr)
    if plr ~= LocalPlayer then targetDropdown:Add(plr.DisplayName) end
end)

Killer:AddSwitch("Kill Selected Targets", function(state)
    task.spawn(function()
        while state do
            local char = LocalPlayer.Character
            if char and #targetPlayerNames > 0 then
                for _, name in ipairs(targetPlayerNames) do
                    local plr = Players:FindFirstChild(name)
                    if plr and plr.Character then
                        local root = plr.Character:FindFirstChild("HumanoidRootPart")
                        if root then
                            muscleEvent:FireServer("punch", "rightHand")
                            muscleEvent:FireServer("punch", "leftHand")
                        end
                    end
                end
            end
            task.wait(0.02)
        end
    end)
end)

--// Auto Punch - No Animation (Best Method 2026)
Killer:AddSwitch("Auto Punch [No Animation - OP]", function(state)
    task.spawn(function()
        while state do
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
            task.wait(0.008)  -- very fast but stable
        end
    end)
end)

--// Auto Equip Punch
Killer:AddSwitch("Auto Equip Punch", function(state)
    task.spawn(function()
        while state do
            local punch = LocalPlayer.Backpack:FindFirstChild("Punch")
            if punch then punch.Parent = LocalPlayer.Character end
            task.wait(0.2)
        end
    end)
end)

--// Remove Punch Animation
Killer:AddButton("Remove Punch Animation", function()
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

--// Auto Slams
Killer:AddSwitch("Auto Ground Slams", function(state)
    task.spawn(function()
        while state do
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
end)

--// Good Mode (Brawl)
Killer:AddSwitch("Good Mode (Brawl Loop)", function(state)
    task.spawn(function()
        while state do
            ReplicatedStorage.rEvents.brawlEvent:FireServer("joinBrawl")
            task.wait(1)
        end
    end)
end)

--// Combo NaN
Killer:AddButton("Combo NaN (Size Glitch)", function()
    ReplicatedStorage.rEvents.changeSpeedSizeRemote:InvokeServer("changeSize", 0/0)
end)

--// Follow / TP Behind
local followDropdown = Killer:AddDropdown("Follow / TP Behind Player", function(displayName)
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr.DisplayName == displayName then
            followTarget = plr.Name
            following = true
            break
        end
    end
end)

for _, plr in ipairs(Players:GetPlayers()) do
    if plr ~= LocalPlayer then followDropdown:Add(plr.DisplayName) end
end

Killer:AddButton("Stop Following", function()
    following = false
    followTarget = nil
end)

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

--// Time Changer
local times = {"Morning","Noon","Afternoon","Sunset","Night","Midnight","Dawn","Early Morning"}
local timeDrop = Killer:AddDropdown("Change Time", function(sel)
    Lighting.Brightness = 2
    Lighting.FogEnd = 100000
    if sel == "Morning" then Lighting.ClockTime = 6 Lighting.Ambient = Color3.fromRGB(200,200,255)
    elseif sel == "Noon" then Lighting.ClockTime = 12 Lighting.Brightness = 3 Lighting.Ambient = Color3.fromRGB(255,255,255)
    elseif sel == "Afternoon" then Lighting.ClockTime = 16 Lighting.Ambient = Color3.fromRGB(255,220,180)
    elseif sel == "Sunset" then Lighting.ClockTime = 18 Lighting.Ambient = Color3.fromRGB(255,150,100) Lighting.FogEnd = 500
    elseif sel == "Night" then Lighting.ClockTime = 20 Lighting.Brightness = 1.5 Lighting.Ambient = Color3.fromRGB(100,100,150)
    elseif sel == "Midnight" then Lighting.ClockTime = 0 Lighting.Brightness = 1 Lighting.Ambient = Color3.fromRGB(50,50,100)
    elseif sel == "Dawn" then Lighting.ClockTime = 4 Lighting.Ambient = Color3.fromRGB(180,180,220)
    elseif sel == "Early Morning" then Lighting.ClockTime = 2 Lighting.Brightness = 1.2 Lighting.Ambient = Color3.fromRGB(100,120,180)
    end
end)

for _, t in ipairs(times) do timeDrop:Add(t) end

print("✅ Muscle Legends Kill Tab Fixed & Loaded!")
