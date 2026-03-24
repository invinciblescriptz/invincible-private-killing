-- INV Public || Made by Invincible || (Cleaned & Fixed 2026 Edition)

-- Load UI Library
local success, library = pcall(function()
    return loadstring(game:HttpGet("https://raw.githubusercontent.com/memejames/elerium-v2-ui-library/main/Library", true))()
end)

if not success or not library then
    warn("[INV] Failed to load UI Library")
    return
end

-- Create Main Window
local window = library:AddWindow("INV Public || Made by Invincible ||", {
    main_color = Color3.fromRGB(0, 0, 0),
    min_size = Vector2.new(520, 660),
    can_resize = false,
})

-- Tabs
local tabInfo = window:AddTab("Information")
local tabMain = window:AddTab("Main")
local tabTeleport = window:AddTab("Teleport")
local tabRebirth = window:AddTab("Rebirth")
-- Removed the "Kill" tab

-- Info tab content
tabInfo:AddLabel("Invincible likes Ninja Turtles").TextSize = 24
tabInfo:AddLabel("Made by Invincible").TextSize = 22
tabInfo:AddLabel("More features coming 🔜").TextSize = 22

-- Services & Globals
local Players = game:GetService("Players")
local RS = game:GetService("RelocatedStorage")
local WS = game:GetService("Workspace")
local UIS = game:GetService("UserInputService")
local Run = game:GetService("RunService")
local VU = game:GetService("VirtualUser")
local Lighting = game:GetService("Lighting")
local LP = Players.LocalPlayer

local muscleEvent = LP:WaitForChild("muscleEvent", 15)
if not muscleEvent then
    warn("[INV] muscleEvent not found after 15s — are you in Lifting Legends?")
    return
end

getgenv().NexusRunning = true -- Global flag for script running

-- State variables
local states = {
    AutoPushups = false,
    AutoWeight = false,
    AutoHandstands = false,
    FastPunch = false,
    InfiniteJump = false,
    AntiAFK = false,
    KillAura = false,
    AutoSlam = false,
}

-- Additional globals
getgenv().rockFarmActive = false
getgenv().selectedRockDurability = 0
getgenv().AutoSpinWheel = false
getgenv().FastPackFarm = false
getgenv().AutoTPMuscleKing = false
getgenv().PositionLocked = false

-- Helper functions
local function getHRP()
    local char = LP.Character
    return char and char:FindFirstChild("HumanoidRootPart")
end

local function tpTo(pos)
    local hrp = getHRP()
    if hrp then
        hrp.CFrame = CFrame.new(pos)
    end
end

local function safeEquip(toolName)
    pcall(function()
        local char = LP.Character
        if not char or not char:FindFirstChild("Humanoid") then return end
        local t = LP.Backpack:FindFirstChild(toolName) or char:FindFirstChild(toolName)
        if t and t.Parent ~= char then
            char.Humanoid:EquipTool(t)
        end
    end)
end

-- Auto-Rep (repetition) logic
local function startAutoRep(toolName, key)
    task.spawn(function()
        while states[key] and getgenv().NexusRunning do
            pcall(function()
                local char = LP.Character
                local h = char and char:FindFirstChildOfClass("Humanoid")
                if not h or h.Health <= 0 then
                    task.wait(2)
                    return
                end
                safeEquip(toolName)
                muscleEvent:FireServer("rep")
            end)
            task.wait(0.1)
        end
        -- Cleanup: unequip tool
        pcall(function()
            local t = LP.Character and LP.Character:FindFirstChild(toolName)
            if t then t.Parent = LP.Backpack end
        end)
    end)
end

-- Infinite Jump
local jumpConn
local function toggleInfiniteJump(enabled)
    states.InfiniteJump = enabled
    if enabled then
        if jumpConn then return end
        jumpConn = UIS.JumpRequest:Connect(function()
            pcall(function()
                local hum = LP.Character and LP.Character:FindFirstChildOfClass("Humanoid")
                if hum then
                    hum:ChangeState(Enum.HumanoidStateType.Jumping)
                end
            end)
        end)
    else
        if jumpConn then
            jumpConn:Disconnect()
            jumpConn = nil
        end
    end
end

-- Anti-AFK
local afkConn
local function toggleAntiAFK(enabled)
    states.AntiAFK = enabled
    if enabled then
        if afkConn then return end
        afkConn = LP.Idled:Connect(function()
            pcall(function()
                VU:CaptureController()
                VU:ClickButton2(Vector2.new())
            end)
        end)
    else
        if afkConn then
            afkConn:Disconnect()
            afkConn = nil
        end
    end
end

-- Rock Farm Loop
task.spawn(function()
    while getgenv().NexusRunning do
        if getgenv().rockFarmActive and getgenv().selectedRockDurability > 0 then
            pcall(function()
                local char = LP.Character
                local h = char and char:FindFirstChildOfClass("Humanoid")
                if not h or h.Health <= 0 then task.wait(1.4) return end

                local durVal = LP:FindFirstChild("Durability")
                if not durVal or durVal.Value < getgenv().selectedRockDurability then
                    task.wait(0.6)
                    return
                end

                safeEquip("Punch")
                local folder = WS:FindFirstChild("machinesFolder")
                if folder then
                    for _, obj in ipairs(folder:GetDescendants()) do
                        if obj.Name == "neededDurability" and obj:IsA("IntValue") and obj.Value == getgenv().selectedRockDurability then
                            local rock = obj.Parent:FindFirstChild("Rock")
                            if rock and rock:IsA("BasePart") then
                                local hrp = getHRP()
                                if hrp then
                                    firetouchinterest(rock, hrp, 0)
                                    task.wait(0.025)
                                    firetouchinterest(rock, hrp, 1)
                                    muscleEvent:FireServer("punch", "leftHand")
                                    muscleEvent:FireServer("punch", "rightHand")
                                end
                            end
                        end
                    end
                end
            end)
        end
        task.wait(0.22)
    end
end)

-- ==================== -- Main Tab Controls
local toolsFolder = tabMain:AddFolder("Tools")
local rockFarmFolder = tabMain:AddFolder("Rock Farm")

-- Auto features toggles
toolsFolder:AddSwitch("Auto Pushups", function(s)
    states.AutoPushups = s
    if s then startAutoRep("Pushups", "AutoPushups") end
end)

toolsFolder:AddSwitch("Auto Weight", function(s)
    states.AutoWeight = s
    if s then startAutoRep("Weight", "AutoWeight") end
end)

toolsFolder:AddSwitch("Auto Handstands", function(s)
    states.AutoHandstands = s
    if s then startAutoRep("Handstands", "AutoHandstands") end
end)

toolsFolder:AddSwitch("Fast Punch", function(s)
    states.FastPunch = s
end)

toolsFolder:AddSwitch("Infinite Jump", function(s)
    toggleInfiniteJump(s)
end)

toolsFolder:AddSwitch("Anti-AFK", function(s)
    toggleAntiAFK(s)
end)

-- Unlock Gamepass (client-side)
tabInfo:AddButton("Unlock Auto-Lift Gamepass", function()
    pcall(function()
        local ids = RS:FindFirstChild("gamepassIds")
        if not ids then warn("[INV] gamepassIds not found") return end
        local owned = LP:FindFirstChild("ownedGamepasses")
        if not owned then
            owned = Instance.new("Folder", LP)
            owned.Name = "ownedGamepasses"
        end
        for _, v in ipairs(ids:GetChildren()) do
            if v:IsA("IntValue") and not owned:FindFirstChild(v.Name) then
                local nv = Instance.new("IntValue", owned)
                nv.Name = v.Name
                nv.Value = v.Value
            end
        end
    end)
end)

-- Teleport Buttons
local teleportLocations = {
    {"Spawn / Safe", Vector3.new(-6, 5, -196)},
    {"Secret Area", Vector3.new(1947, 2, 6191)},
    {"Tiny Area", Vector3.new(-39, 6, 1886)},
    {"Frozen", Vector3.new(-2623, 6, -409)},
    {"Mythical", Vector3.new(2251, 6, 1073)},
    {"Inferno", Vector3.new(-6759, 6, -1285)},
    {"Legend", Vector3.new(4603, 990, -3898)},
    {"Muscle King", Vector3.new(-8626, 16, -5730)},
    {"Jungle", Vector3.new(-8642, 6, 2381)},
    {"Lava Brawl", Vector3.new(4471, 119, -8836)},
    {"Desert Brawl", Vector3.new(960, 17, -7398)},
    {"Beach Brawl", Vector3.new(-1849, 20, -6335)},
}

for _, v in ipairs(teleportLocations) do
    tabTeleport:AddButton(v[1], function() tpTo(v[2]) end)
end

-- Rock Farm Controls
local function setRockFarm(durability, enable)
    if enable then
        getgenv().selectedRockDurability = durability
        getgenv().rockFarmActive = true
    else
        if getgenv().selectedRockDurability == durability then
            getgenv().rockFarmActive = false
            getgenv().selectedRockDurability = 0
        end
    end
end

local rockFarmOptions = {
    {"Farm Tiny Island Rock (0)", 0},
    {"Farm Starter Island Rock (100)", 100},
    {"Farm Legend Beach Rock (5000)", 5000},
    {"Farm Frost Gym Rock (150k)", 150000},
    {"Farm Mythical Gym Rock (400k)", 400000},
    {"Farm Eternal Gym Rock (750k)", 750000},
    {"Farm Legend Gym Rock (1M)", 1000000},
    {"Farm Muscle King Gym Rock (5M)", 5000000},
    {"Farm Ancient Jungle Rock (10M)", 10000000},
}

for _, option in ipairs(rockFarmOptions) do
    local name, dur = option[1], option[2]
    rockFarmFolder:AddSwitch(name, function(s) setRockFarm(dur, s) end)
end

rockFarmFolder:AddButton("Stop All Rock Farming", function()
    getgenv().rockFarmActive = false
    getgenv().selectedRockDurability = 0
end)

-- Additional Bug Fix in Rock Farming Loop
task.spawn(function()
    while getgenv().NexusRunning do
        if getgenv().rockFarmActive and getgenv().selectedRockDurability > 0 then
            pcall(function()
                local char = LP.Character
                local h = char and char:FindFirstChildOfClass("Humanoid")
                if not h or h.Health <= 0 then task.wait(1.4) return end

                local dur = LP:FindFirstChild("Durability")
                if not dur or dur.Value < getgenv().selectedRockDurability then
                    task.wait(0.6)
                    return
                end

                safeEquip("Punch")

                local folder = WS:FindFirstChild("machinesFolder")
                if folder then
                    for _, obj in pairs(folder:GetDescendants()) do
                        if obj.Name == "neededDurability" and obj:IsA("IntValue") and obj.Value == getgenv().selectedRockDurability then
                            local rock = obj.Parent:FindFirstChild("Rock")
                            if rock and rock:IsA("BasePart") then
                                local hrp = getHRP()
                                if hrp then
                                    if char:FindFirstChild("RightHand") then
                                        firetouchinterest(rock, hrp, 0)
                                        task.wait(0.03)
                                        firetouchinterest(rock, hrp, 1)
                                    end
                                    if char:FindFirstChild("LeftHand") then
                                        firetouchinterest(rock, hrp, 0)
                                        task.wait(0.03)
                                        firetouchinterest(rock, hrp, 1)
                                    end
                                    -- Punch
                                    muscleEvent:FireServer("punch", "leftHand")
                                    muscleEvent:FireServer("punch", "rightHand")
                                end
                            end
                        end
                    end
                end
            end)
        end
        task.wait(0.18)
    end
end)

-- ==================== -- Rebirth Tab Controls
local autoRebirth = false
local autoSize = false

tabRebirth:AddSwitch("Auto Rebirth (Infinite)", function(val)
    autoRebirth = val
    if val then
        task.spawn(function()
            while autoRebirth and getgenv().NexusRunning do
                pcall(function()
                    RS.rEvents.rebirthRemote:InvokeServer("rebirthRequest")
                end)
                task.wait(0.45)
            end
        end)
    end
end)

tabRebirth:AddSwitch("Auto Size 1", function(val)
    autoSize = val
    if val then
        task.spawn(function()
            while autoSize and getgenv().NexusRunning do
                pcall(function()
                    RS.rEvents.changeSpeedSizeRemote:InvokeServer("changeSize", 1)
                end)
                task.wait(0.2)
            end
        end)
    end
end)

-- Auto TP to Muscle King
tabRebirth:AddSwitch("Auto TP → Muscle King", function(v)
    getgenv().AutoTPMuscleKing = v
    if v then
        task.spawn(function()
            while getgenv().AutoTPMuscleKing do
                tpTo(Vector3.new(-8646, 17, -5738))
                task.wait(0.7)
            end
        end)
    end
end)

-- Pack Farming (Fast)
tabRebirth:AddLabel("Pack Farming").TextSize = 22
tabRebirth:AddSwitch("Fast Rebirth Pack Farm", function(v)
    getgenv().FastPackFarm = v
    if v then
        task.spawn(function()
            local rep = RS.rEvents
            local petsFolder = LP:WaitForChild("petsFolder")
            local leader = LP:WaitForChild("leaderstats")

            local function equipAll()
                for _, folder in ipairs(petsFolder:GetChildren()) do
                    if folder:IsA("Folder") then
                        for _, pet in ipairs(folder:GetChildren()) do
                            rep.equipPetEvent:FireServer("equipPet", pet)
                        end
                    end
                end
            end

            while getgenv().FastPackFarm do
                pcall(function()
                    equipAll()
                    task.wait(0.2)

                    local need = 10000 + 5000 * leader.Rebirths.Value
                    if LP:FindFirstChild("ultimatesFolder") and LP.ultimatesFolder:FindFirstChild("Golden Rebirth") then
                        need = math.floor(need * (1 - LP.ultimatesFolder["Golden Rebirth"].Value * 0.1))
                    end

                    -- Equip best pet fallback
                    local bestPet = petsFolder:FindFirstChild("Swift Samurai")
                    if bestPet then
                        rep.equipPetEvent:FireServer("equipPet", bestPet)
                    end

                    while leader.Strength.Value < need do
                        for _ = 1, 12 do muscleEvent:FireServer("rep") end
                        task.wait(0.05)
                    end

                    -- Equip stronger pet
                    local strongPet = petsFolder:FindFirstChild("Tribal Overlord")
                    if strongPet then
                        rep.equipPetEvent:FireServer("equipPet", strongPet)
                    end

                    -- Jungle Bar Sit
                    local bar = WS.machinesFolder and WS.machinesFolder:FindFirstChild("Jungle Bar Lift")
                    if bar and bar:FindFirstChild("interactSeat") then
                        getHRP().CFrame = bar.interactSeat.CFrame * CFrame.new(0, 4, 0)
                        task.wait(0.4)
                    end

                    -- Rebirth
                    local oldRebirth = leader.Rebirths.Value
                    repeat
                        rep.rebirthRemote:InvokeServer("rebirthRequest")
                        task.wait(0.15)
                    until leader.Rebirths.Value > oldRebirth or not getgenv().FastPackFarm
                end)
                task.wait(0.6)
            end
        end)
    end
end)

-- Optimization Buttons
tabRebirth:AddButton("Full Optimization", function()
    pcall(function()
        -- Remove GUI
        for _, g in ipairs(LP.PlayerGui:GetChildren()) do
            if g:IsA("ScreenGui") then g:Destroy() end
        end
        -- Remove particles & lights
        for _, o in ipairs(WS:GetDescendants()) do
            if o:IsA("ParticleEmitter") or o:IsA("PointLight") then o:Destroy() end
        end
        -- Skybox & Lighting
        for _, s in ipairs(Lighting:GetChildren()) do
            if s:IsA("Sky") then s:Destroy() end
        end
        local darkSky = Instance.new("Sky", Lighting)
        for _, face in ipairs({"Bk", "Dn", "Ft", "Lf", "Rt", "Up"}) do
            darkSky["Skybox" .. face] = "rbxassetid://0"
        end
        Lighting.Brightness = 0
        Lighting.ClockTime = 0
        Lighting.OutdoorAmbient = Color3.new()
        Lighting.Ambient = Color3.new()
        Lighting.FogEnd = 120
    end)
end)

tabRebirth:AddButton("Anti-Lag (Performance)", function()
    pcall(function()
        settings().Rendering.QualityLevel = Enum.QualityLevel.Level01
        Lighting.GlobalShadows = false
        Lighting.FogEnd = 1e9
        Lighting.Brightness = 0
        for _, o in ipairs(WS:GetDescendants()) do
            if o:IsA("ParticleEmitter") then o.Enabled = false end
            if o:IsA("Decal") or o:IsA("Texture") then o.Transparency = 1 end
        end
    end)
end)

-- Quick TPs
tabRebirth:AddButton("Jungle Squat Area", function()
    tpTo(Vector3.new(-8371.4, 6.8, 2858.9))
end)
tabRebirth:AddButton("Jungle Lift Area", function()
    tpTo(Vector3.new(-8652.9, 29.3, 2089.3))
end)

-- Position Lock
local lockConn
tabRebirth:AddSwitch("Lock Position", function(en)
    getgenv().PositionLocked = en
    local hrp = getHRP()
    if not hrp then return end

    if en then
        local pos = hrp.Position
        hrp.Anchored = true
        if lockConn then lockConn:Disconnect() end
        lockConn = Run.Heartbeat:Connect(function()
            if getgenv().PositionLocked and hrp and hrp.Parent then
                hrp.CFrame = CFrame.new(pos)
            end
        end)
    else
        if hrp then hrp.Anchored = false end
        if lockConn then lockConn:Disconnect() lockConn = nil end
    end
end)

-- ======================== -- Updated "Auto Punch (Fast)" Feature
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
