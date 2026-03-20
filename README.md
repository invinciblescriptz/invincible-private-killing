local PlayerService = game:GetService("Players")
local ReplicatedStorageService = game:GetService("ReplicatedStorage")
local LightingService = game:GetService("Lighting")
local SoundService = game:GetService("SoundService")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local LocalPlayer = PlayerService.LocalPlayer
local CharacterModel = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()

local ScriptLibrary = loadstring(game:HttpGet("https://raw.githubusercontent.com/p4020854-hub/Lb/refs/heads/main/X", true))()
local MainWindow = ScriptLibrary:AddWindow(string.format("Invincible 2.0 | Hello %s", LocalPlayer.DisplayName), {
    min_size = Vector2.new(680, 870),
    can_resize = true,
    main_color = Color3.fromRGB(0, 0, 0)
})

-- Tabs
local MainTab = MainWindow:AddTab("Main")
local FarmTab = MainWindow:AddTab("Farm Op")
local KillTab = MainWindow:AddTab("Kill")
local TeleportTab = MainWindow:AddTab("Teleport")
local MusicTab = MainWindow:AddTab("Music")
local CreditsTab = MainWindow:AddTab("Credits")

-- Global variables
_G.InfiniteJump = false
_G.AutoSpinWheel = false
_G.AutoRepFarmEnabled = false
local AutoEatEgg = false
local AntiFling = false
local LockPosition = false
local ShowPets = false
local ShowOtherPets = false
local WalkOnWater = false
local MusicUrl = ""
local CurrentSound = nil
_G.autoGoodKarma = false
_G.autoBadKarma = false
_G.fastHitActive = false
_G.whitelistedPlayers = _G.whitelistedPlayers or {}
_G.viewPlayer = false
_G.ViewTarget = ""
_G.autoWhitelistFriends = false
_G.selectedPet = ""

-- Main Tab - Settings
MainTab:AddLabel("Important:").TextSize = 22

local AntiFlingSwitch = MainTab:AddSwitch("Anti Fling", function(SwitchState)
    AntiFling = SwitchState
end)
AntiFlingSwitch:Set(true)

local LockPositionSwitch = MainTab:AddSwitch("Lock Position", function(SwitchState)
    LockPosition = SwitchState
end)
LockPositionSwitch:Set(false)

local ShowPetsSwitch = MainTab:AddSwitch("Show Pets", function(SwitchState)
    ShowPets = SwitchState
end)
ShowPetsSwitch:Set(false)

local ShowOtherPetsSwitch = MainTab:AddSwitch("Show Other Pets", function(SwitchState)
    ShowOtherPets = SwitchState
end)
ShowOtherPetsSwitch:Set(false)

MainTab:AddLabel("Misc:").TextSize = 22

local InfiniteJumpSwitch = MainTab:AddSwitch("Infinite Jump", function(SwitchState)
    _G.InfiniteJump = SwitchState
    if SwitchState then
        UserInputService.JumpRequest:Connect(function()
            if _G.InfiniteJump and CharacterModel and CharacterModel:FindFirstChildOfClass("Humanoid") then
                CharacterModel.Humanoid:ChangeState("Jumping")
            end
        end)
    end
end)
InfiniteJumpSwitch:Set(false)

local WalkOnWaterSwitch = MainTab:AddSwitch("Walk on Water", function(SwitchState)
    WalkOnWater = SwitchState
end)
WalkOnWaterSwitch:Set(true)

local TimeDropdown = MainTab:AddDropdown("Change Time", function(SelectedOption)
    if SelectedOption == "Night" then
        LightingService.ClockTime = 0
    elseif SelectedOption == "Day" then
        LightingService.ClockTime = 12
    elseif SelectedOption == "Midnight" then
        LightingService.ClockTime = 0
        LightingService.Brightness = 0
    end
end)
TimeDropdown:Add("Night")
TimeDropdown:Add("Day")
TimeDropdown:Add("Midnight")

-- Farm Op Tab
local StrengthSwitch = FarmTab:AddSwitch("Strength Op", function(SwitchState)
    _G.AutoRepFarmEnabled = SwitchState
    warn("[Auto Rep Farm] Estado cambiado a:", SwitchState and "ON" or "OFF")
end)
StrengthSwitch:Set(false)

local AutoEatSwitch = FarmTab:AddSwitch("Auto Eat Egg 30 Minutes", function(SwitchState)
    AutoEatEgg = SwitchState
    task.spawn(function()
        while AutoEatEgg do
            local Backpack = LocalPlayer:FindFirstChild("Backpack")
            if Backpack then
                local EggItem = Backpack:FindFirstChild("Protein Egg")
                if EggItem then
                    -- Add logic if needed
                end
            end
            task.wait(1800) -- Wait 30 minutes
        end
    end)
end)
AutoEatSwitch:Set(false)

local SpinWheelSwitch = FarmTab:AddSwitch("Spin Fortune Wheel", function(SwitchState)
    _G.AutoSpinWheel = SwitchState
    if SwitchState then
        task.spawn(function()
            while _G.AutoSpinWheel do
                -- Add spin wheel logic here
                task.wait(0.1)
            end
        end)
    end
end)
SpinWheelSwitch:Set(false)

FarmTab:AddSwitch("Hide All Frames", function(SwitchState)
    -- Implement hide all frames if needed
end)

FarmTab:AddButton("Anti Lag", function()
    for _, DescendantObject in pairs(workspace:GetDescendants()) do
        if DescendantObject:IsA("ParticleEmitter") then
            DescendantObject.Enabled = false
        end
    end
    LightingService.GlobalShadows = false
    LightingService.FogEnd = 9e9
    LightingService.Brightness = 0
    settings().Rendering.QualityLevel = 1
    for _, DecalObject in pairs(workspace:GetDescendants()) do
        if DecalObject:IsA("Decal") then
            DecalObject.Transparency = 1
        end
    end
    for _, EffectObject in pairs(LightingService:GetChildren()) do
        if EffectObject:IsA("BlurEffect") then
            EffectObject.Enabled = false
        end
    end
    game.StarterGui:SetCore("SendNotification", {
        Text = "Full optimization applied!",
        Title = "Anti Lag activado",
        Duration = 5
    })
end)

FarmTab:AddButton("Full Optimization", function()
    for _, GuiObject in pairs(LocalPlayer:WaitForChild("PlayerGui"):GetChildren()) do
        if GuiObject:IsA("ScreenGui") then
            GuiObject:Destroy()
        end
    end
    for _, ObjectInstance in pairs(workspace:GetDescendants()) do
        if ObjectInstance:IsA("ParticleEmitter") or ObjectInstance:IsA("PointLight") then
            ObjectInstance:Destroy()
        end
    end
    for _, Skybox in pairs(LightingService:GetChildren()) do
        if Skybox:IsA("Sky") then
            Skybox:Destroy()
        end
    end
    local DarkSkybox = Instance.new("Sky")
    DarkSkybox.Name = "DarkSky"
    DarkSkybox.SkyboxBk = "rbxassetid://0"
    DarkSkybox.SkyboxDn = "rbxassetid://0"
    DarkSkybox.SkyboxFt = "rbxassetid://0"
    DarkSkybox.SkyboxLf = "rbxassetid://0"
    DarkSkybox.SkyboxRt = "rbxassetid://0"
    DarkSkybox.SkyboxUp = "rbxassetid://0"
    DarkSkybox.Parent = LightingService
    LightingService.Brightness = 0
    LightingService.ClockTime = 0
    LightingService.OutdoorAmbient = Color3.new(0, 0, 0)
    LightingService.Ambient = Color3.new(0, 0, 0)
    LightingService.FogColor = Color3.new(0, 0, 0)
    LightingService.FogEnd = 100
end)

FarmTab:AddButton("Equip Swift Samurai", function()
    print("Botón presionado: equipando 8 Swift Samurai")
    local PetsContainer = LocalPlayer:FindFirstChild("petsFolder")
    if PetsContainer then
        -- Add logic to equip pets if needed
    end
end)

FarmTab:AddButton("Jungle Squat", function()
    if CharacterModel and CharacterModel:FindFirstChild("HumanoidRootPart") then
        CharacterModel.HumanoidRootPart.CFrame = CFrame.new(-8371.4336, 6.7981, 2858.8853)
    end
end)

FarmTab:AddButton("Jungle lift", function()
    if CharacterModel and CharacterModel:FindFirstChild("HumanoidRootPart") then
        CharacterModel.HumanoidRootPart.CFrame = CFrame.new(-8652.8672, 29.2667, 2089.2617)
    end
end)

-- Rebirths info
local RebirthsFolder = FarmTab:AddFolder("Fast Rebirths Functions")
RebirthsFolder:AddLabel("0d 0h 0m 0s").TextSize = 18
RebirthsFolder:AddLabel("Rebirths: 0").TextSize = 18
RebirthsFolder:AddLabel("Rebirths Gained: 0").TextSize = 18

-- Rebirths tracking
task.spawn(function()
    while true do
        pcall(function()
            local LeaderboardStats = LocalPlayer:FindFirstChild("leaderstats")
            if LeaderboardStats then
                local RebirthCount = LeaderboardStats:FindFirstChild("Rebirths")
                if RebirthCount then
                    -- Update display if needed
                end
            end
        end)
        task.wait(1)
    end
end)

-- Create ground base parts
task.spawn(function()
    local BasePosition = Vector3.new(-2, -9.5, -2)
    local BlockSize = Vector3.new(2048, 1, 2048)
    for X = -5, 5 do
        for Z = -5, 5 do
            local Position = BasePosition + Vector3.new(X * 2048, 0, Z * 2048)
            local Part = Instance.new("Part")
            Part.Size = BlockSize
            Part.Position = Position
            Part.Anchored = true
            Part.Transparency = 1
            Part.CanCollide = true
            Part.Name = string.format("FloorPart_%d_%d", X, Z)
            Part.Parent = workspace
        end
    end
end)

-- Kill Tab - Player control
local function UpdatePlayerLists()
    WhitelistDropdown:Clear()
    ViewDropdown:Clear()
    for _, Player in ipairs(PlayerService:GetPlayers()) do
        if Player ~= LocalPlayer then
            WhitelistDropdown:Add(Player.DisplayName)
            ViewDropdown:Add(Player.DisplayName)
        end
    end
end

PlayerService.PlayerAdded:Connect(UpdatePlayerLists)
PlayerService.PlayerRemoving:Connect(UpdatePlayerLists)
task.wait(1)
UpdatePlayerLists()

local PetDropdown = KillTab:AddDropdown("Select Pet", function(Value)
    _G.selectedPet = Value
end)
PetDropdown:Add("Wild Wizard")
PetDropdown:Add("Mighty Monster")

local AutoGoodKarmaSwitch = KillTab:AddSwitch("Auto Good Karma", function(BooleanValue)
    _G.autoGoodKarma = BooleanValue
    task.spawn(function()
        while _G.autoGoodKarma do
            pcall(function()
                local Character = LocalPlayer.Character
                if Character then
                    local GoodKarmaTool = Character:FindFirstChild("GoodKarma") or LocalPlayer.Backpack:FindFirstChild("GoodKarma")
                    if GoodKarmaTool then
                        Character.Humanoid:EquipTool(GoodKarmaTool)
                        GoodKarmaTool:Activate()
                    end
                end
            end)
            task.wait(0.1)
        end
    end)
end)
AutoGoodKarmaSwitch:Set(false)

local AutoWhitelistSwitch = KillTab:AddSwitch("Auto Whitelist Friends", function(BooleanValue)
    _G.autoWhitelistFriends = BooleanValue
    task.spawn(function()
        while _G.autoWhitelistFriends do
            pcall(function()
                for _, Player in ipairs(PlayerService:GetPlayers()) do
                    if Player ~= LocalPlayer and LocalPlayer:IsFriendsWith(Player.UserId) then
                        if not table.find(_G.whitelistedPlayers, Player.Name) then
                            table.insert(_G.whitelistedPlayers, Player.Name)
                        end
                    end
                end
            end)
            task.wait(5)
        end
    end)
end)
AutoWhitelistSwitch:Set(false)

local WhitelistDropdown = KillTab:AddDropdown("Add to Whitelist", function(DisplayName)
    for _, Player in ipairs(PlayerService:GetPlayers()) do
        if Player.DisplayName == DisplayName then
            if not table.find(_G.whitelistedPlayers, Player.Name) then
                table.insert(_G.whitelistedPlayers, Player.Name)
            end
            break
        end
    end
end)

local AutoPunchSwitch = KillTab:AddSwitch("Auto Punch", function(BooleanValue)
    _G.fastHitActive = BooleanValue
    task.spawn(function()
        while _G.fastHitActive do
            pcall(function()
                local Character = LocalPlayer.Character
                if Character then
                    local PunchTool = Character:FindFirstChild("Punch") or LocalPlayer.Backpack:FindFirstChild("Punch")
                    if PunchTool then
                        Character.Humanoid:EquipTool(PunchTool)
                        PunchTool:Activate()
                    end
                end
            end)
            task.wait(0.1)
        end
    end)
end)
AutoPunchSwitch:Set(false)

KillTab:AddButton("Remove Punch Anim", function()
    local Character = LocalPlayer.Character
    if Character and Character:FindFirstChild("Humanoid") then
        local Animator = Character.Humanoid:FindFirstChild("Animator")
        if Animator then
            for _, Track in pairs(Animator:GetPlayingAnimationTracks()) do
                if Track.Animation.AnimationId == "rbxassetid://3638729053" or Track.Animation.AnimationId == "rbxassetid://3638767427" then
                    Track:Stop()
                    Track:Destroy()
                end
            end
        end
    end
end)

local ViewDropdown = KillTab:AddDropdown("Select View Target", function(Value)
    _G.ViewTarget = Value
end)

local ViewPlayerSwitch = KillTab:AddSwitch("View Player", function(BooleanValue)
    _G.viewPlayer = BooleanValue
    task.spawn(function()
        while _G.viewPlayer do
            pcall(function()
                local Target = nil
                for _, Player in pairs(PlayerService:GetPlayers()) do
                    if Player.DisplayName == _G.ViewTarget then
                        Target = Player
                        break
                    end
                end

                if Target and Target.Character and Target.Character:FindFirstChild("Humanoid") then
                    workspace.CurrentCamera.CameraSubject = Target.Character.Humanoid
                else
                    workspace.CurrentCamera.CameraSubject = LocalPlayer.Character.Humanoid
                end
            end)
            task.wait(0.1)
        end
        workspace.CurrentCamera.CameraSubject = LocalPlayer.Character.Humanoid
    end)
end)
ViewPlayerSwitch:Set(false)

-- Size Buttons
KillTab:AddButton("Size 30", function()
    local Arguments = { [1] = "changeSize", [2] = 30 }
    ReplicatedStorageService:WaitForChild("rEvents"):WaitForChild("changeSpeedSizeRemote"):InvokeServer(unpack(Arguments))
end)

KillTab:AddButton("Size 2", function()
    local Arguments = { [1] = "changeSize", [2] = 2 }
    ReplicatedStorageService:WaitForChild("rEvents"):WaitForChild("changeSpeedSizeRemote"):InvokeServer(unpack(Arguments))
end)

-- Change Time Dropdown
local TimeDropdown2 = KillTab:AddDropdown("Change Time", function(Value)
    if Value == "Midnight" then LightingService.ClockTime = 0
    elseif Value == "Morning" then LightingService.ClockTime = 8
    elseif Value == "Noon" then LightingService.ClockTime = 12
    elseif Value == "Afternoon" then LightingService.ClockTime = 14
    elseif Value == "Sunset" then LightingService.ClockTime = 18
    elseif Value == "Night" then LightingService.ClockTime = 20
    elseif Value == "Dawn" then LightingService.ClockTime = 6
    elseif Value == "Early Morning" then LightingService.ClockTime = 5
    end
end)
local Times2 = {"Morning", "Noon", "Afternoon", "Sunset", "Night", "Midnight", "Dawn", "Early Morning"}
for _, TimeOption in ipairs(Times2) do TimeDropdown2:Add(TimeOption) end

-- Blacklist System
local BlacklistFileName = "GenesisBlacklist_" .. LocalPlayer.Name .. ".txt"
if not isfile(BlacklistFileName) then
    writefile(BlacklistFileName, "")
end

KillTab:AddTextBox("Add to Blacklist", function(TextParameter)
    local CurrentBlacklist = isfile(BlacklistFileName) and readfile(BlacklistFileName) or ""
    appendfile(BlacklistFileName, "," .. TextParameter)
end, {["placeholder"] = "Ex: User1, User2"})

-- Music Tab
local function PlayMusic()
    if MusicUrl ~= "" then
        local MusicFileName = "GenesisMusic_1.mp3"
        writefile(MusicFileName, game:HttpGet(MusicUrl))
        if CurrentSound then CurrentSound:Destroy() end
        CurrentSound = Instance.new("Sound")
        CurrentSound.Name = "GenesisMP3Sound"
        CurrentSound.Parent = SoundService
        CurrentSound.SoundId = getcustomasset(MusicFileName)
        CurrentSound.Volume = 1
        CurrentSound.Looped = false
        CurrentSound:Play()
    end
end

MusicTab:AddLabel("🎵 00:00 / 00:00")
MusicTab:AddTextBox("MP3 URL", function(ValueParameter)
    MusicUrl = ValueParameter
end, {["clear"] = false})

MusicTab:AddButton("Play", PlayMusic)

MusicTab:AddButton("Stop", function()
    if CurrentSound then CurrentSound:Stop() end
end)

MusicTab:AddTextBox("Volume (0-5)", function(ParameterValue)
    local NumberValue = tonumber(ParameterValue)
    if NumberValue and CurrentSound then
        CurrentSound.Volume = math.clamp(NumberValue, 0, 5)
    end
end, {["clear"] = false})

-- Teleport Tab
TeleportTab:AddButton("Spawn", function()
    if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
        LocalPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(2, 8, 115)
    end
end)

TeleportTab:AddButton("Secret Area", function()
    if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
        LocalPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(1947, 2, 6191)
    end
end)

TeleportTab:AddButton("Tiny Island", function()
    if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
        LocalPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(-34, 7, 1903)
    end
end)

TeleportTab:AddButton("Frozen Island", function()
    if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
        LocalPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(-2600, 3.6, -403)
    end
end)

TeleportTab:AddButton("Mythical Island", function()
    if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
        LocalPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(2255, 7, 1071)
    end
end)

TeleportTab:AddButton("Hell Island", function()
    if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
        LocalPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(-6768, 7, -1287)
    end
end)

TeleportTab:AddButton("Legend Island", function()
    if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
        LocalPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(4604, 991, -3887)
    end
end)

TeleportTab:AddButton("Muscle King Island", function()
    if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
        LocalPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(-8646, 17, -5738)
    end
end)

TeleportTab:AddButton("Jungle Island", function()
    if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
        LocalPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(-8659, 6, 2384)
    end
end)

TeleportTab:AddButton("Brawl Lava", function()
    if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
        LocalPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(4471, 119, -8836)
    end
end)
