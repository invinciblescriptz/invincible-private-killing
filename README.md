local ScriptLibrary = loadstring(game:HttpGet("https://raw.githubusercontent.com/p4020854-hub/Lb/refs/heads/main/X", true))()
local MainWindow = ScriptLibrary:AddWindow(string.format("Genesis Hub | Hello %s", LocalPlayer.DisplayName), {
    min_size = Vector2.new(680, 870),
    can_resize = true,
    main_color = Color3.fromRGB(0, 0, 0)
})

-- Simple Kill Aura (auto punch nearby players)
getgenv().KillAuraEnabled = false
getgenv().KillRange = 15       -- adjust distance
getgenv().PunchDelay = 0.05    -- lower = faster (but can lag/kick risk)

local player = game.Players.LocalPlayer
local mouse = player:GetMouse()

local function getClosest()
    local closest, dist = nil, KillRange
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
        wait(PunchDelay)
        if not KillAuraEnabled then continue end
        
        local target = getClosest()
        if target and target.Character and target.Character:FindFirstChild("Humanoid") and target.Character.Humanoid.Health > 0 then
            -- Most common remote in Muscle Legends for punch/attack
            game:GetService("ReplicatedStorage").Events.Punch:FireServer(target.Character.Humanoid)
            -- Some older scripts use: game.ReplicatedStorage.rEvents.punch:FireServer()
        end
    end
end)

-- Add toggle button to your GUI
local tab = window:AddTab("Combat")
tab:AddSwitch("Kill Aura", function(bool)
    getgenv().KillAuraEnabled = bool
end)

tab:AddSlider("Kill Range", 5, 50, 15, function(val)
    getgenv().KillRange = val
end)
