local ScriptLibrary = loadstring(game:HttpGet("https://raw.githubusercontent.com/p4020854-hub/Lb/refs/heads/main/X", true))()
local MainWindow = ScriptLibrary:AddWindow(string.format("invincible private killing || Hello %s", game.Players.LocalPlayer.DisplayName), {
    ["min_size"] = Vector2.new(660, 520),
    ["can_resize"] = true,
    ["main_color"] = Color3.fromRGB(255, 192, 203) -- Pink color
})

local Players = game:GetService("Players")
local LP = Players.LocalPlayer
local workspace = game:GetService("Workspace")
local Lighting = game:GetService("Lighting")
local VirtualUser = game:GetService("VirtualUser")
local muscleEvent = game:GetService("ReplicatedStorage"):WaitForChild("muscleEvent") -- Make sure this exists

local window = MainWindow

-- Anti AFK
local antiAfkConn
local states = {}
local function toggleAntiAfk(state)
    states.AntiAFK = state
    if state then
        if antiAfkConn then return end
        antiAfkConn = LP.Idled:Connect(function()
            pcall(function()
                VirtualUser:CaptureController()
                VirtualUser:ClickButton2(Vector2.new())
            end)
        end)
    else
        if antiAfkConn then antiAfkConn:Disconnect() antiAfkConn = nil end
    end
end

-- =================== Tabs and Features ===================

-- Main Kill Tab
local KillerTab = window:AddTab("Kill")
local Kill = KillerTab

-- Auto Karma
Kill:AddSwitch("Auto Good Karma", function(state)
    AutoGoodKarma = state
    task.spawn(function()
        while AutoGoodKarma do
            local char = LP.Character
            local rightHand = char and char:FindFirstChild("RightHand")
            local leftHand = char and char:FindFirstChild("LeftHand")
            if char and rightHand and leftHand then
                for _, target in ipairs(Players:GetPlayers()) do
                    if target ~= LP then
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
                    if target ~= LP then
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

-- Auto Whitelist friends
local playerWhitelist = {}
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

local autoKill = false
Kill:AddSwitch("Auto Kill", function(state)
    autoKill = state
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

-- Target selection dropdown
local targetPlayerNames = {}
local targetDropdown = Kill:AddDropdown("Select Target", function(displayName)
    for _, player in ipairs(Players:GetPlayers()) do
        if player.DisplayName == displayName then
            if not table.find(targetPlayerNames, player.Name) then
                table.insert(targetPlayerNames, player.Name)
            end
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

Players.PlayerAdded:Connect(function()
    updateTargetDropdown()
end)
Players.PlayerRemoving:Connect(function(player)
    updateTargetDropdown()
    for i = #targetPlayerNames, 1, -1 do
        if targetPlayerNames[i] == player.Name then
            table.remove(targetPlayerNames, i)
        end
    end
end)

local killTargetActive = false
Kill:AddSwitch("Start Kill Target", function(state)
    killTargetActive = state
    task.spawn(function()
        while killTargetActive do
            local char = LP.Character or LP.CharacterAdded:Wait()
            local rightHand = char and char:FindFirstChild("RightHand")
            local leftHand = char and char:FindFirstChild("LeftHand")
            local punch = LP.Backpack:FindFirstChild("Punch")
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

-- View and follow
local ViewDropdownItems = {}
local ViewTargetName = nil
local spying = false

local viewDropdown = Kill:AddDropdown("Select View Target", function(value)
    ViewTargetName = value
end)

for _, player in ipairs(Players:GetPlayers()) do
    if player ~= LP then
        viewDropdown:Add(player.DisplayName)
        ViewDropdownItems[player.Name] = player.DisplayName
    end
end

Players.PlayerAdded:Connect(function(player)
    if player ~= LP then
        viewDropdown:Add(player.DisplayName)
        ViewDropdownItems[player.Name] = player.DisplayName
    end
end)

Players.PlayerRemoving:Connect(function(player)
    if player ~= LP then
        ViewDropdownItems[player.Name] = nil
        -- rebuild dropdown
        local options = {}
        for _, display in pairs(ViewDropdownItems) do
            options[#options + 1] = display
        end
        viewDropdown:Clear()
        for _, displayName in ipairs(options) do
            viewDropdown:Add(displayName)
        end
    end
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

local following = false

Kill:AddSwitch("View Player", function(state)
    spying = state
    if not spying then
        local cam = workspace.CurrentCamera
        cam.CameraSubject = LP.Character and LP.Character:FindFirstChild("Humanoid") or LP
        return
    end
    task.spawn(function()
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
    end)
end)

Kill:AddButton("Stop Following", function()
    spying = false
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

-- =================== Additional kill method tab ===================

local killmethodTab = window:AddTab("kill method")

-- Fast Punch toggle
local function toggleFastPunch(s)
    states.FastPunch = s
    if s then
        task.spawn(function()
            while states.FastPunch do
                pcall(function()
                    print("Fast Punch: firing") -- Debug
                    if muscleEvent then
                        muscleEvent:FireServer("punch", "rightHand")
                        muscleEvent:FireServer("punch", "leftHand")
                    else
                        warn("muscleEvent not found")
                    end
                    local char = LP.Character
                    if char then
                        local punch = char:FindFirstChild("Punch") or LP.Backpack:FindFirstChild("Punch")
                        if punch and punch.Parent ~= char then
                            -- Equip the punch tool
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
end

killmethodTab:AddSwitch("Fast Punch", toggleFastPunch)

-- Auto Slam toggle
local autoSlamActive = false
killmethodTab:AddSwitch("auto slams", function(state)
    autoSlamActive = state
    if state then
        task.spawn(function()
            while autoSlamActive do
                print("Auto Slam: attempting") -- Debug
                local player = LP
                local groundSlam = player.Backpack:FindFirstChild("Ground Slam") or (player.Character and player.Character:FindFirstChild("Ground Slam"))
                if groundSlam then
                    if groundSlam.Parent == player.Backpack then
                        groundSlam.Parent = player.Character
                    end
                    if groundSlam:FindFirstChild("attackTime") then
                        groundSlam.attackTime.Value = 0
                    end
                    print("Firing slam event") -- Debug
                    player.muscleEvent:FireServer("slam")
                    groundSlam:Activate()
                else
                    warn("Ground Slam not found")
                end
                task.wait(0.1)
            end
        end)
    end
end)

-- Change time dropdown
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

local function changeTime(selection)
    -- Reset
    Lighting.Brightness = 2
    Lighting.FogEnd = 100000
    Lighting.Ambient = Color3.fromRGB(127,127,127)

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
end

local timeDropdown = killmethodTab:AddDropdown("change time", changeTime)
for _, option in ipairs(timeOptions) do
    timeDropdown:Add(option)
end

-- =================== END ===================
