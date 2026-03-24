-- Load external ScriptLibrary
local ScriptLibrary = loadstring(game:HttpGet("https://raw.githubusercontent.com/p4020854-hub/Lb/refs/heads/main/X", true))()

-- Assuming LocalPlayer and Players are globally available
local Players = game:GetService("Players")
local LP = Players.LocalPlayer

-- Create the main window
local MainWindow = ScriptLibrary:AddWindow(
    string.format("Invincible Private Killing || Hello %s", LP.DisplayName),
    {
        ["min_size"] = Vector2.new(400, 870),
        ["can_resize"] = true,
        ["main_color"] = Color3.fromRGB(255, 192, 203)
    }
)

local KillerTab = MainWindow:AddTab("Killer") -- Create Killer tab

-- Variables for toggles
local AutoGoodKarma = false
local AutoBadKarma = false
local AutoPunch = false
local AutoKill = false
local spying = false
local killTargetActive = false
local ViewTargetName = nil
local states = {}

-- Utility to get players
local function getPlayers()
    return Players:GetPlayers()
end

-- Auto Good Karma
KillerTab:AddSwitch("Auto Good Karma", function(state)
    AutoGoodKarma = state
    if state then
        task.spawn(function()
            while AutoGoodKarma do
                local char = LP.Character
                local rightHand = char and char:FindFirstChild("RightHand")
                local leftHand = char and char:FindFirstChild("LeftHand")
                if char and rightHand and leftHand then
                    for _, target in ipairs(getPlayers()) do
                        if target ~= LP and target.Character then
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

-- Auto Bad Karma
KillerTab:AddSwitch("Auto Bad Karma", function(state)
    AutoBadKarma = state
    if state then
        task.spawn(function()
            while AutoBadKarma do
                local char = LP.Character
                local rightHand = char and char:FindFirstChild("RightHand")
                local leftHand = char and char:FindFirstChild("LeftHand")
                if char and rightHand and leftHand then
                    for _, target in ipairs(getPlayers()) do
                        if target ~= LP and target.Character then
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

-- Auto Whitelist Friends
local playerWhitelist = {}
KillerTab:AddSwitch("Auto Whitelist Friends", function(state)
    if state then
        for _, player in ipairs(getPlayers()) do
            if player ~= LP and LP:IsFriendsWith(player.UserId) then
                playerWhitelist[player.Name] = true
            end
        end
        Players.PlayerAdded:Connect(function(player)
            if LP:IsFriendsWith(player.UserId) then
                playerWhitelist[player.Name] = true
            end
        end)
    else
        for name in pairs(playerWhitelist) do
            playerWhitelist[name] = nil
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

-- Auto Punch (Fast)
KillerTab:AddSwitch("Auto Punch (Fast)", function(s)
    states.FastPunch = s
    if s then
        task.spawn(function()
            while states.FastPunch and getgenv().NexusRunning do
                pcall(function()
                    if muscleEvent then
                        muscleEvent:FireServer("punch", "rightHand")
                        muscleEvent:FireServer("punch", "leftHand")
                    end
                    local char = LP.Character
                    if char then
                        local punch = char:FindFirstChild("Punch") or LP.Backpack:FindFirstChild("Punch")
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
end)

-- Auto Kill
local autoKill = false
KillerTab:AddSwitch("Auto Kill", function(state)
    autoKill = state
    if state then
        task.spawn(function()
            while autoKill do
                local char = LP.Character or LP.CharacterAdded:Wait()
                local rightHand = char and char:FindFirstChild("RightHand")
                local leftHand = char and char:FindFirstChild("LeftHand")
                local punch = LP.Backpack:FindFirstChild("Punch")
                if punch and not char:FindFirstChild("Punch") then
                    punch.Parent = char
                end
                if rightHand and leftHand then
                    for _, target in ipairs(getPlayers()) do
                        if target ~= LP and not playerWhitelist[target.Name] then
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

-- Target dropdown for specific target
local targetPlayerNames = {}
local SelectedTarget = nil

local function updateTargetDropdown()
    local options = {}
    for _, player in ipairs(getPlayers()) do
        if player ~= LP then
            table.insert(options, player.DisplayName)
        end
    end
    -- Assuming you have a dropdown UI element
    -- For example:
    -- targetDropdown:Clear()
    -- for _, name in ipairs(options) do
    --     targetDropdown:Add(name)
    -- end
end

local targetDropdown = KillerTab:AddDropdown("Select Target", function(displayName)
    for _, player in ipairs(getPlayers()) do
        if player.DisplayName == displayName then
            SelectedTarget = player.Name
            break
        end
    end
end)

-- Initialize dropdown options
updateTargetDropdown()

Players.PlayerAdded:Connect(function(player)
    updateTargetDropdown()
end)
Players.PlayerRemoving:Connect(function(player)
    updateTargetDropdown()
end)

-- Start kill target feature
KillerTab:AddSwitch("Start Kill Target", function(state)
    killTargetActive = state
    if state then
        task.spawn(function()
            while killTargetActive do
                local char = LP.Character or LP.CharacterAdded:Wait()
                local rightHand = char and char:FindFirstChild("RightHand")
                local leftHand = char and char:FindFirstChild("LeftHand")
                local punch = LP.Backpack:FindFirstChild("Punch")
                if punch and not char:FindFirstChild("Punch") then
                    punch.Parent = char
                end
                for _, name in ipairs(targetPlayerNames) do
                    local target = Players:FindFirstChild(name)
                    if target and target ~= LP and target.Character then
                        local rootPart = target.Character:FindFirstChild("HumanoidRootPart")
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

-- View and follow other players
local ViewDropdownItems = {}
local function updateViewDropdown()
    local options = {}
    for _, player in ipairs(getPlayers()) do
        if player ~= LP then
            table.insert(options, player.DisplayName)
            ViewDropdownItems[player.Name] = player.DisplayName
        end
    end
    -- Assuming you have a dropdown UI, update it here
    -- viewDropdown:Clear()
    -- for _, displayName in ipairs(options) do
    --     viewDropdown:Add(displayName)
    -- end
end

local ViewTargetName = nil
local spying = false

local viewDropdown = KillerTab:AddDropdown("Select View Target", function(value)
    ViewTargetName = value
end)

updateViewDropdown()

Players.PlayerAdded:Connect(function(player)
    updateViewDropdown()
end)
Players.PlayerRemoving:Connect(function(player)
    ViewDropdownItems[player.Name] = nil
    updateViewDropdown()
end)

local function followPlayer(target)
    local myChar = LP.Character
    local targetChar = target.Character
    if not (myChar and targetChar) then return end
    local myHRP = myChar:FindFirstChild("HumanoidRootPart")
    local targetHRP = targetChar:FindFirstChild("HumanoidRootPart")
    if myHRP and targetHRP then
        local followPos = targetHRP.Position - (targetHRP.CFrame.LookVector * 3)
        myHRP.CFrame = CFrame.new(followPos, targetHRP.Position)
    end
end

local function startFollowing()
    while spying do
        local target = Players:FindFirstChild(ViewTargetName)
        if target and target ~= LP then
            local humanoid = target.Character and target.Character:FindFirstChild("Humanoid")
            if humanoid then
                workspace.CurrentCamera.CameraSubject = humanoid
            end
        end
        task.wait(0.1)
    end
end

-- Toggle follow
KillerTab:AddSwitch("View Player", function(state)
    spying = state
    if state then
        task.spawn(startFollowing)
    else
        workspace.CurrentCamera.CameraSubject = LP.Character and LP.Character:FindFirstChild("Humanoid") or LP
    end
end)

-- Button to stop following
KillerTab:AddButton("Stop Following", function()
    spying = false
    workspace.CurrentCamera.CameraSubject = LP.Character and LP.Character:FindFirstChild("Humanoid") or LP
    print("Stopped following")
end)

-- Remove punch animation
KillerTab:AddButton("Remove Punch Anim", function()
    local character = LP.Character
    if character and character:FindFirstChild("Humanoid") then
        local humanoid = character:FindFirstChild("Humanoid")
        for _, track in pairs(humanoid:GetPlayingAnimationTracks()) do
            if track.Animation and 
               (track.Animation.AnimationId == "rbxassetid://3638729053" or 
                track.Animation.AnimationId == "rbxassetid://3638767427") then
                track:Stop()
            end
        end
    end
end)
