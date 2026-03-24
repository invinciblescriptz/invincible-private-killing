local ScriptLibrary = loadstring(game:HttpGet("https://raw.githubusercontent.com/p4020854-hub/Lb/refs/heads/main/X", true))()
local MainWindow = ScriptLibrary:AddWindow(string.format("invincible private killing || Hello %s", LocalPlayer.DisplayName), {
    ["min_size"] = Vector2.new(470, 660),
    ["can_resize"] = true,
    ["main_color"] = Color3.fromRGB(255, 192, 203) -- Pink color
})

local Players = game:GetService("Players")
local LP = Players.LocalPlayer
local Workspace = game:GetService("Workspace")
local Lighting = game:GetService("Lighting")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local window = MainWindow

local KillerTab = window:AddTab("Kill")
local toolsFolder = KillerTab:AddFolder("Tools")

local states = {}
local AutoGoodKarma, AutoBadKarma, autoKill = false, false, false
local SelectedTarget = nil
local playerWhitelist = {}
local ViewTargetName = nil
local spying = false

-- Auto Punch (Fast)
toolsFolder:AddSwitch("Auto Punch (Fast)", function(s)
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

-- Boost FPS (Lag Fix)
toolsFolder:AddButton("Boost FPS (Lag Fix)", function()
    Lighting.GlobalShadows = false
    Lighting.Brightness = 1
    Lighting.ClockTime = 12
    Lighting.FogEnd = 9999
    Lighting.FogStart = 0
    for _, v in pairs(Workspace:GetDescendants()) do
        if v:IsA("ParticleEmitter") or v:IsA("Smoke") or v:IsA("Fire") or v:IsA("Sparkles") or
           v:IsA("BloomEffect") or v:IsA("BlurEffect") or v:IsA("DepthOfFieldEffect") or v:IsA("SunRaysEffect") then
            v.Enabled = false
        end
    end
end)

-- Auto Lift Gamepass
toolsFolder:AddButton("Auto Lift Gamepass (Free)", function()
    pcall(function()
        local gpFolder = ReplicatedStorage:FindFirstChild("gamepassIds")
        if not gpFolder then return end
        local owned = LP:FindFirstChild("ownedGamepasses") or Instance.new("Folder")
        owned.Name = "ownedGamepasses"
        owned.Parent = LP
        for _, gp in pairs(gpFolder:GetChildren()) do
            if gp:IsA("IntValue") then
                local val = Instance.new("IntValue")
                val.Name = gp.Name
                val.Value = gp.Value
                val.Parent = owned
            end
        end
    end)
end)

-- Auto Good Karma
KillerTab:AddSwitch("Auto Good Karma", function(state)
    AutoGoodKarma = state
    task.spawn(function()
        while AutoGoodKarma do
            local char = LP.Character
            if char then
                local rightHand = char:FindFirstChild("RightHand")
                local leftHand = char:FindFirstChild("LeftHand")
                if rightHand and leftHand then
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
            end
            task.wait(0.01)
        end
    end)
end)

-- Auto Bad Karma
KillerTab:AddSwitch("Auto Bad Karma", function(state)
    AutoBadKarma = state
    task.spawn(function()
        while AutoBadKarma do
            local char = LP.Character
            if char then
                local rightHand = char:FindFirstChild("RightHand")
                local leftHand = char:FindFirstChild("LeftHand")
                if rightHand and leftHand then
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
            end
            task.wait(0.01)
        end
    end)
end)

-- Auto Whitelist friends
KillerTab:AddSwitch("Auto Whitelist Friends", function(state)
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

-- Auto Kill
KillerTab:AddSwitch("Auto Kill", function(state)
    autoKill = state
    task.spawn(function()
        while autoKill do
            local char = LP.Character or LP.CharacterAdded:Wait()
            local rightHand = char:FindFirstChild("RightHand")
            local leftHand = char:FindFirstChild("LeftHand")
            local punch = LP.Backpack:FindFirstChild("Punch") or nil
            if punch and not char:FindFirstChild("Punch") then
                punch.Parent = char
            end
            if rightHand and leftHand then
                for _, target in ipairs(Players:GetPlayers()) do
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
end)

-- Target dropdown for specific target
local targetPlayerNames = {}
local targetDropdown = KillerTab:AddDropdown("Select Target", function(displayName)
    for _, player in ipairs(Players:GetPlayers()) do
        if player.DisplayName == displayName then
            if not table.find(targetPlayerNames, player.Name) then
                table.insert(targetPlayerNames, player.Name)
            end
            -- Save as selected target
            SelectedTarget = player.Name
            break
        end
    end
end)

local function updateTargetDropdown()
    local options = {}
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LP then
            options[#options + 1] = player.DisplayName
        end
    end
    targetDropdown:Clear()
    for _, name in ipairs(options) do
        targetDropdown:Add(name)
    end
end

-- Refresh player list dynamically
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

local killTargetActive = false
KillerTab:AddSwitch("Start Kill Target", function(state)
    killTargetActive = state
    task.spawn(function()
        while killTargetActive do
            local char = LP.Character or LP.CharacterAdded:Wait()
            local rightHand = char:FindFirstChild("RightHand")
            local leftHand = char:FindFirstChild("LeftHand")
            local punch = LP.Backpack:FindFirstChild("Punch") or nil
            if punch and not char:FindFirstChild("Punch") then
                punch.Parent = char
            end
            if rightHand and leftHand then
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
            end
            task.wait(0.05)
        end
    end)
end)

-- View and follow other players
local ViewDropdownItems = {}
local function populateViewDropdown()
    local options = {}
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LP then
            ViewDropdownItems[player.Name] = player.DisplayName
            options[#options + 1] = player.DisplayName
        end
    end
    -- Clear and add options
    viewDropdown:Clear()
    for _, displayName in ipairs(options) do
        viewDropdown:Add(displayName)
    end
end

local viewDropdown = KillerTab:AddDropdown("Select View Target", function(value)
    ViewTargetName = value
end)

populateViewDropdown()

Players.PlayerAdded:Connect(function(player)
    if player ~= LP then
        populateViewDropdown()
    end
end)

Players.PlayerRemoving:Connect(function(player)
    if player ~= LP then
        populateViewDropdown()
        -- Remove from list
        for i, name in ipairs(targetPlayerNames) do
            if name == player.Name then
                table.remove(targetPlayerNames, i)
            end
        end
    end
end)

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

KillerTab:AddSwitch("View Player", function(state)
    spying = state
    if not spying then
        local cam = workspace.CurrentCamera
        cam.CameraSubject = LP.Character and LP.Character:FindFirstChild("Humanoid") or LP
        return
    end
    task.spawn(function()
        while spying do
            if ViewTargetName then
                local target = Players:FindFirstChild(ViewTargetName)
                if target and target ~= LP then
                    local humanoid = target.Character and target.Character:FindFirstChild("Humanoid")
                    if humanoid then
                        workspace.CurrentCamera.CameraSubject = humanoid
                    end
                end
            end
            task.wait(0.1)
        end
    end)
end)

-- Button to stop following
KillerTab:AddButton("Stop Following", function()
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
KillerTab:AddButton("Remove Punch Anim", function()
    local character = LP.Character
    if character then
        local humanoid = character:FindFirstChild("Humanoid")
        if humanoid then
            for _, track in pairs(humanoid:GetPlayingAnimationTracks()) do
                if track.Animation then
                    local id = track.Animation.AnimationId
                    if id == "rbxassetid://3638729053" or id == "rbxassetid://3638767427" then
                        track:Stop()
                    end
                end
            end
        end
    end
end)
