local ScriptLibrary = loadstring(game:HttpGet("https://raw.githubusercontent.com/p4020854-hub/Lb/refs/heads/main/X", true))()
local MainWindow = ScriptLibrary:AddWindow(string.format("invincible private killing || Hello %s", game.Players.LocalPlayer.DisplayName), {
    ["min_size"] = Vector2.new(400, 870),
    ["can_resize"] = true,
    ["main_color"] = Color3.fromRGB(255, 192, 203) -- Pink color
})

local Players = game:GetService("Players")
local LP = Players.LocalPlayer

local window = MainWindow -- Adjusted for consistency

local KillerTab = window:AddTab("Killer") -- Create the tab for killer features

-- Variables to hold toggle states
local AutoGoodKarma, AutoBadKarma, autoKill, spying, killTargetActive
local SelectedTarget = nil
local playerWhitelist = {}
local ViewTargetName = nil

-- Auto Good Karma
Kill:AddSwitch("Auto Good Karma", function(state)
    AutoGoodKarma = state
    task.spawn(function()
        while AutoGoodKarma do
            local char = LP.Character
            local rightHand = char and char:FindFirstChild("RightHand")
            local leftHand = char and char:FindFirstChild("LeftHand")
            if char and rightHand and leftHand then
                for _, target in ipairs(Players:GetPlayers()) do
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
end)

-- Auto Bad Karma
Kill:AddSwitch("Auto Bad Karma", function(state)
    AutoBadKarma = state
    task.spawn(function()
        while AutoBadKarma do
            local char = LP.Character
            local rightHand = char and char:FindFirstChild("RightHand")
            local leftHand = char and char:FindFirstChild("LeftHand")
            if char and rightHand and leftHand then
                for _, target in ipairs(Players:GetPlayers()) do
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
end)

-- Auto Whitelist friends
Kill:AddSwitch("Auto Whitelist Friends", function(state)
    if state then
        for _, player in ipairs(Players:GetPlayers()) do
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

Kill:AddTextBox("Whitelist", function(text)
    local target = Players:FindFirstChild(text)
    if target then
        playerWhitelist[target.Name] = true
    end
end)

Kill:AddTextBox("UnWhitelist", function(text)
    local target = Players:FindFirstChild(text)
    if target then
        playerWhitelist[target.Name] = nil
    end
end)

-- Auto Kill
Kill:AddSwitch("Auto Kill", function(state)
    autoKill = state
    task.spawn(function()
        while autoKill do
            local char = LP.Character or LP.CharacterAdded:Wait()
            local rightHand = char:FindFirstChild("RightHand")
            local leftHand = char:FindFirstChild("LeftHand")
            local punch = LP.Backpack:FindFirstChild("Punch")
            if punch and not char:FindFirstChild("Punch") then
                punch.Parent = char
            end
            if rightHand and leftHand then
                for _, target in ipairs(Players:GetPlayers()) do
                    if target ~= LP and not playerWhitelist[target.Name] and target.Character then
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

-- Target dropdown for specific target
local targetPlayerNames = {}
local SelectedTarget = nil

local function updateTargetDropdown()
    local options = {}
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LP then
            table.insert(options, player.DisplayName)
        end
    end
    -- Clear and rebuild dropdown
    if targetDropdown then
        targetDropdown:Clear()
        for _, name in ipairs(options) do
            targetDropdown:Add(name)
        end
    end
end

local targetDropdown = Kill:AddDropdown("Select Target", function(displayName)
    for _, player in ipairs(Players:GetPlayers()) do
        if player.DisplayName == displayName then
            SelectedTarget = player.Name
            break
        end
    end
end)

Players.PlayerAdded:Connect(function()
    updateTargetDropdown()
end)

Players.PlayerRemoving:Connect(function(player)
    updateTargetDropdown()
    -- Remove from target list if present
    for i = #targetPlayerNames, 1, -1 do
        if targetPlayerNames[i] == player.Name then
            table.remove(targetPlayerNames, i)
        end
    end
end)

-- Start kill target toggle
Kill:AddSwitch("Start Kill Target", function(state)
    killTargetActive = state
    task.spawn(function()
        while killTargetActive do
            local char = LP.Character or LP.CharacterAdded:Wait()
            local rightHand = char:FindFirstChild("RightHand")
            local leftHand = char:FindFirstChild("LeftHand")
            local punch = LP.Backpack:FindFirstChild("Punch")
            if punch and not char:FindFirstChild("Punch") then
                punch.Parent = char
            end
            if rightHand and leftHand then
                for _, name in ipairs(targetPlayerNames) do
                    local target = Players:FindFirstChild(name)
                    if target and target.Character then
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
            end
            task.wait(0.05)
        end
    end)
end)

-- View and follow other players
local ViewDropdownItems = {}
local spying = false

local viewDropdown = Kill:AddDropdown("Select View Target", function(value)
    ViewTargetName = value
end)

local function refreshViewDropdown()
    viewDropdown:Clear()
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LP then
            viewDropdown:Add(player.DisplayName)
            ViewDropdownItems[player.Name] = player.DisplayName
        end
    end
end

refreshViewDropdown()

Players.PlayerAdded:Connect(refreshViewDropdown)
Players.PlayerRemoving:Connect(refreshViewDropdown)

-- Follow selected player
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

local following = false

Kill:AddSwitch("View Player", function(state)
    spying = state
    if not spying then
        workspace.CurrentCamera.CameraSubject = LP.Character and LP.Character:FindFirstChild("Humanoid") or LP
        return
    end
    task.spawn(function()
        while spying do
            local target = Players:FindFirstChild(ViewTargetName)
            if target and target ~= LP and target.Character then
                local humanoid = target.Character:FindFirstChild("Humanoid")
                if humanoid then
                    workspace.CurrentCamera.CameraSubject = humanoid
                end
            end
            task.wait(0.1)
        end
    end)
end)

-- Button to stop following
Kill:AddButton("Stop Following", function()
    spying = false
    workspace.CurrentCamera.CameraSubject = LP.Character and LP.Character:FindFirstChild("Humanoid") or LP
    print("Stopped following")
end)

-- Auto follow loop
task.spawn(function()
    while true do
        task.wait(0.01)
        if spying and ViewTargetName then
            local target = Players:FindFirstChild(ViewTargetName)
            if target then
                followPlayer(target)
            end
        end
    end
end)

-- Remove punch animation
Kill:AddButton("Remove Punch Anim", function()
    local character = LP.Character
    if character and character:FindFirstChild("Humanoid") then
        local humanoid = character:FindFirstChild("Humanoid")
        if humanoid then
            for _, track in pairs(humanoid:GetPlayingAnimationTracks()) do
                if track.Animation and (track.Animation.AnimationId == "rbxassetid://3638729053" or track.Animation.AnimationId == "rbxassetid://3638767427") then
                    track:Stop()
                end
            end
        end
    end
end)
