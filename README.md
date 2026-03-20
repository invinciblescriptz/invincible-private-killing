local ScriptLibrary = loadstring(game:HttpGet("https://raw.githubusercontent.com/p4020854-hub/Lb/refs/heads/main/X", true))()
local MainWindow = ScriptLibrary:AddWindow(string.format("Invincible Private Killing | Hello %s", game.Players.LocalPlayer.DisplayName), {
    min_size = Vector2.new(680, 870),
    can_resize = true,
    main_color = Color3.fromRGB(0, 0, 0)
})

local player = game.Players.LocalPlayer
local ReplicatedStorage = game:GetService("ReplicatedStorage")

-- Globals (expanded)
getgenv().KillAuraEnabled = false
getgenv().KillRange = 15
getgenv().PunchDelay = 0.05
getgenv().AuraMode = "Closest"  -- "Closest" or "Random"
getgenv().SkipTeammates = true   -- skip same team if teams exist

getgenv().AutoKillEnabled = false
getgenv().AutoKillTarget = nil    -- username string

getgenv().FastAttackEnabled = false
getgenv().FastAttackDelay = 0.03

getgenv().ESPEnabled = false

-- Remotes (2026 common paths - test in console if needed)
local punchRemote = ReplicatedStorage:FindFirstChild("Events") and ReplicatedStorage.Events:FindFirstChild("Punch")
                     or ReplicatedStorage:FindFirstChild("rEvents") and ReplicatedStorage.rEvents:FindFirstChild("punch")
                     or ReplicatedStorage:FindFirstChild("rEvents") and ReplicatedStorage.rEvents:FindFirstChild("Punch")
                     or nil

local trainRemote = ReplicatedStorage:FindFirstChild("rEvents") and ReplicatedStorage.rEvents:FindFirstChild("use") or nil
local rebirthRemote = ReplicatedStorage:FindFirstChild("Events") and ReplicatedStorage.Events:FindFirstChild("rebirth") or nil

-- Helper: Get closest or random in range
local function getKillTarget()
    local candidates = {}
    for _, plr in ipairs(game.Players:GetPlayers()) do
        if plr ~= player and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") and plr.Character:FindFirstChild("Humanoid") and plr.Character.Humanoid.Health > 0 then
            local root = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
            if root then
                local mag = (root.Position - plr.Character.HumanoidRootPart.Position).Magnitude
                if mag <= getgenv().KillRange then
                    if getgenv().SkipTeammates and plr.Team and player.Team and plr.Team == player.Team then continue end
                    table.insert(candidates, plr)
                end
            end
        end
    end
    
    if #candidates == 0 then return nil end
    
    if getgenv().AuraMode == "Random" then
        return candidates[math.random(1, #candidates)]
    else
        -- Closest
        table.sort(candidates, function(a,b)
            local da = (player.Character.HumanoidRootPart.Position - a.Character.HumanoidRootPart.Position).Magnitude
            local db = (player.Character.HumanoidRootPart.Position - b.Character.HumanoidRootPart.Position).Magnitude
            return da < db
        end)
        return candidates[1]
    end
end

-- Kill Aura loop
spawn(function()
    while true do
        task.wait(getgenv().PunchDelay)
        if not getgenv().KillAuraEnabled then continue end
        local target = getKillTarget()
        if target and punchRemote then
            punchRemote:FireServer(target.Character.Humanoid)
        end
    end
end)

-- Auto Kill specific target
spawn(function()
    while true do
        task.wait(0.08)
        if not getgenv().AutoKillEnabled or not getgenv().AutoKillTarget then continue end
        local target = game.Players:FindFirstChild(getgenv().AutoKillTarget)
        if target and target.Character and target.Character:FindFirstChild("Humanoid") and target.Character.Humanoid.Health > 0 and punchRemote then
            punchRemote:FireServer(target.Character.Humanoid)
        end
    end
end)

-- Fast Attack (spam punch no target check - very risky)
spawn(function()
    while true do
        task.wait(getgenv().FastAttackDelay)
        if not getgenv().FastAttackEnabled or not punchRemote then continue end
        punchRemote:FireServer()  -- some servers accept empty for self-punch/spam
    end
end)

-- Simple ESP (highlight players red)
local function createESP(plr)
    if plr == player or not plr.Character then return end
    local highlight = Instance.new("Highlight")
    highlight.FillColor = Color3.fromRGB(255, 0, 0)
    highlight.OutlineColor = Color3.fromRGB(255, 255, 0)
    highlight.FillTransparency = 0.5
    highlight.OutlineTransparency = 0
    highlight.Parent = plr.Character
    plr.CharacterAdded:Connect(function(char)
        highlight.Adornee = char
    end)
end

spawn(function()
    while getgenv().ESPEnabled do
        task.wait(1)
        for _, plr in ipairs(game.Players:GetPlayers()) do
            if plr ~= player and plr.Character then
                if not plr.Character:FindFirstChildOfClass("Highlight") then
                    createESP(plr)
                end
            end
        end
    end
end)

-- GUI Tabs (expanded Combat)
local combatTab = MainWindow:AddTab("Combat")

combatTab:AddSwitch("Kill Aura", function(bool)
    getgenv().KillAuraEnabled = bool
end)

combatTab:AddDropdown("Aura Mode", {"Closest", "Random"}, getgenv().AuraMode, function(val)
    getgenv().AuraMode = val
end)

combatTab:AddSwitch("Skip Teammates", function(bool)
    getgenv().SkipTeammates = bool
end):SetState(true)

combatTab:AddSlider("Kill Range", 5, 50, getgenv().KillRange, function(val)
    getgenv().KillRange = val
end)

combatTab:AddSlider("Punch Delay", 0.01, 0.5, getgenv().PunchDelay, function(val)
    getgenv().PunchDelay = val
end, 2)

-- New: Auto Kill target selector
local playerList = {}
for _, plr in ipairs(game.Players:GetPlayers()) do
    if plr ~= player then table.insert(playerList, plr.Name) end
end

combatTab:AddDropdown("Auto Kill Target", playerList, "", function(val)
    getgenv().AutoKillTarget = val
end)

combatTab:AddSwitch("Auto Kill Selected", function(bool)
    getgenv().AutoKillEnabled = bool
end)

combatTab:AddSwitch("Fast Attack Spam", function(bool)
    getgenv().FastAttackEnabled = bool
end)

combatTab:AddSlider("Fast Attack Delay", 0.01, 0.2, getgenv().FastAttackDelay, function(val)
    getgenv().FastAttackDelay = val
end, 3)

combatTab:AddSwitch("Player ESP (Red)", function(bool)
    getgenv().ESPEnabled = bool
    if not bool then
        for _, plr in ipairs(game.Players:GetPlayers()) do
            if plr.Character and plr.Character:FindFirstChildOfClass("Highlight") then
                plr.Character:FindFirstChildOfClass("Highlight"):Destroy()
            end
        end
    end
end)

-- Keep your existing Farming / Movement / Misc tabs here...
-- (paste the rest from previous script: Auto Train, Auto Rebirth, Anti-AFK, WalkSpeed, JumpPower, Safe TP)

local miscTab = MainWindow:AddTab("Misc")
-- ... your existing switches/sliders/buttons ...

-- Extra kill-related button: TP to anti-kill safe spot
local antiKillCFrame = CFrame.new(0, 200, 0)  -- high up, adjust to real safe coords
miscTab:AddButton("Teleport to Anti-Kill Zone", function()
    if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
        player.Character.HumanoidRootPart.CFrame = antiKillCFrame
    end
end)

-- Optional: Fake godmode (local only, visual)
miscTab:AddButton("Fake Godmode (local)", function()
    if player.Character and player.Character:FindFirstChild("Humanoid") then
        player.Character.Humanoid.MaxHealth = math.huge
        player.Character.Humanoid.Health = math.huge
    end
end)

print("Genesis Hub - Enhanced Kill Features Loaded! Use wisely.")
