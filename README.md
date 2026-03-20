local ScriptLibrary = loadstring(game:HttpGet("https://raw.githubusercontent.com/p4020854-hub/Lb/refs/heads/main/X", true))()
local MainWindow = ScriptLibrary:AddWindow(string.format("INVINCIBLE PRIVATE KILL | Hello %s", game.Players.LocalPlayer.DisplayName), {
    min_size = Vector2.new(680, 870),
    can_resize = true,
    main_color = Color3.fromRGB(0, 0, 0)
})

local player = game.Players.LocalPlayer
local mouse = player:GetMouse()
local ReplicatedStorage = game:GetService("ReplicatedStorage")

-- Globals
getgenv().KillAuraEnabled = false
getgenv().KillRange = 15
getgenv().PunchDelay = 0.05

getgenv().AutoTrainEnabled = false
getgenv().TrainType = "Punch"  -- Common: "Punch", "Situps", "Pushups", "Handstands", "Weights"

getgenv().AutoRebirthEnabled = false
getgenv().MinStrengthForRebirth = 1000000  -- 1M strength

getgenv().AntiAFKEnabled = true

-- Find common remotes (2026-era Muscle Legends often uses rEvents or Events)
local punchRemote = ReplicatedStorage:FindFirstChild("Events") and ReplicatedStorage.Events:FindFirstChild("Punch")
                     or ReplicatedStorage:FindFirstChild("rEvents") and ReplicatedStorage.rEvents:FindFirstChild("punch")
                     or nil

local trainRemote = ReplicatedStorage:FindFirstChild("rEvents") and ReplicatedStorage.rEvents:FindFirstChild("use")
                    or ReplicatedStorage:FindFirstChild("Events") and ReplicatedStorage.Events:FindFirstChild("use")
                    or ReplicatedStorage:FindFirstChild("train") or nil

local rebirthRemote = ReplicatedStorage:FindFirstChild("Events") and ReplicatedStorage.Events:FindFirstChild("rebirth")
                      or ReplicatedStorage:FindFirstChild("rEvents") and ReplicatedStorage.rEvents:FindFirstChild("Rebirth")
                      or nil

-- Kill Aura (auto punch closest player)
local function getClosest()
    local closest = nil
    local dist = getgenv().KillRange
    for _, plr in ipairs(game.Players:GetPlayers()) do
        if plr ~= player and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
            local magnitude = (player.Character.HumanoidRootPart.Position - plr.Character.HumanoidRootPart.Position).Magnitude
            if magnitude < dist then
                closest = plr
                dist = magnitude
            end
        end
    end
    return closest
end

spawn(function()
    while true do
        task.wait(getgenv().PunchDelay)
        if not getgenv().KillAuraEnabled then continue end
        local target = getClosest()
        if target and target.Character and target.Character:FindFirstChild("Humanoid") and target.Character.Humanoid.Health > 0 then
            if punchRemote then
                punchRemote:FireServer(target.Character.Humanoid)
            elseif ReplicatedStorage:FindFirstChild("rEvents") and ReplicatedStorage.rEvents:FindFirstChild("punch") then
                ReplicatedStorage.rEvents.punch:FireServer()
            end
        end
    end
end)

-- Auto Train (auto use training machines)
spawn(function()
    while true do
        task.wait(0.12)  -- Slightly safer delay
        if not getgenv().AutoTrainEnabled then continue end
        if trainRemote then
            trainRemote:FireServer(getgenv().TrainType)
        end
    end
end)

-- Auto Rebirth
spawn(function()
    while true do
        task.wait(5)
        if not getgenv().AutoRebirthEnabled then continue end
        local leaderstats = player:FindFirstChild("leaderstats")
        if leaderstats and leaderstats:FindFirstChild("Strength") and leaderstats.Strength.Value >= getgenv().MinStrengthForRebirth then
            if rebirthRemote then
                rebirthRemote:FireServer()
            end
        end
    end
end)

-- Anti-AFK
spawn(function()
    while true do
        task.wait(60)
        if getgenv().AntiAFKEnabled then
            local vu = game:GetService("VirtualUser")
            vu:CaptureController()
            vu:ClickButton2(Vector2.new())
        end
    end
end)

-- GUI Tabs
local combatTab = MainWindow:AddTab("Combat")
combatTab:AddSwitch("Kill Aura", function(bool)
    getgenv().KillAuraEnabled = bool
end)
combatTab:AddSlider("Kill Range", 5, 50, getgenv().KillRange, function(val)
    getgenv().KillRange = val
end)
combatTab:AddSlider("Punch Delay", 0.01, 0.5, getgenv().PunchDelay, function(val)
    getgenv().PunchDelay = val
end, 2)  -- 2 decimal places

local farmingTab = MainWindow:AddTab("Farming")
farmingTab:AddSwitch("Auto Train", function(bool)
    getgenv().AutoTrainEnabled = bool
end)
farmingTab:AddDropdown("Train Type", {"Punch", "Situps", "Pushups", "Handstands", "Weights"}, getgenv().TrainType, function(val)
    getgenv().TrainType = val
end)
farmingTab:AddSwitch("Auto Rebirth", function(bool)
    getgenv().AutoRebirthEnabled = bool
end)
farmingTab:AddTextBox("Min Strength for Rebirth", tostring(getgenv().MinStrengthForRebirth), function(text)
    local num = tonumber(text)
    if num then getgenv().MinStrengthForRebirth = num end
end)

local movementTab = MainWindow:AddTab("Movement")
local char = player.Character or player.CharacterAdded:Wait()
local humanoid = char:WaitForChild("Humanoid")

movementTab:AddSlider("Walk Speed", 16, 300, humanoid.WalkSpeed, function(val)
    humanoid.WalkSpeed = val
end)
movementTab:AddSlider("Jump Power", 50, 500, humanoid.JumpPower, function(val)
    humanoid.JumpPower = val
end)

local miscTab = MainWindow:AddTab("Misc")
miscTab:AddSwitch("Anti AFK", function(bool)
    getgenv().AntiAFKEnabled = bool
end):SetState(true)

-- Example safe zone teleport (adjust CFrame to a real safe spot like high up or spawn)
local safeZoneCFrame = CFrame.new(0, 100, 0)  -- placeholder; find real coords in-game
miscTab:AddButton("Teleport to Safe Zone", function()
    if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
        player.Character.HumanoidRootPart.CFrame = safeZoneCFrame
    end
end)

-- Optional: Character respawn handler (re-apply speed/jump if needed)
player.CharacterAdded:Connect(function(newChar)
    humanoid = newChar:WaitForChild("Humanoid")
    -- You can re-apply sliders here if you want persistent values
end)

print("Genesis Hub loaded! Enjoy responsibly.")
