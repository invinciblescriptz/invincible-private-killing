-- Services
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local Lighting = game:GetService("Lighting")
local Workspace = game:GetService("Workspace")
local LocalPlayer = Players.LocalPlayer

-- Create Main GUI
local gui = Instance.new("ScreenGui", LocalPlayer:WaitForChild("PlayerGui"))
gui.Name = "MyFeatureUI"

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 520, 0, 600)
frame.Position = UDim2.new(0.5, -260, 0.5, -300)
frame.BackgroundColor3 = Color3.fromRGB(0, 10, 0)
frame.BorderSizePixel = 0

-- Make GUI draggable
local function makeDraggable(f)
    local dragging = false
    local dragStart, startPos
    f.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            dragStart = input.Position
            startPos = f.Position
            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then
                    dragging = false
                end
            end)
        end
    end)
    UserInputService.InputChanged:Connect(function(input)
        if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
            local delta = input.Position - dragStart
            f.Position = UDim2.new(
                startPos.X.Scale,
                startPos.X.Offset + delta.X,
                startPos.Y.Scale,
                startPos.Y.Offset + delta.Y
            )
        end
    end)
end

makeDraggable(frame)

-- Main "Kill" Tab
local KillTab = gui:FindFirstChild("Kill") or Instance.new("Folder", gui)
KillTab.Name = "Kill"

local playerWhitelist = {}
local targetPlayerNames = {}
local autoGoodKarma = false
local autoBadKarma = false
local autoKill = false
local killTarget = false
local spying = false
local autoEquipPunch = false
local autoPunchNoAnim = false

local availableTargets = {}
local targetDropdownItems = {}

-- Add Label
local titleLabel = Instance.new("TextLabel", KillTab)
titleLabel.Text = "Select Damage or Durability Pet"
titleLabel.TextSize = 18
titleLabel.Font = Enum.Font.Merriweather
titleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
titleLabel.Size = UDim2.new(0, 200, 0, 30)
titleLabel.Position = UDim2.new(0, 10, 0, 10)

-- Dropdown for selecting pet
local function createDropdown(parent, label, callback)
    local dropdownFrame = Instance.new("Frame", parent)
    dropdownFrame.Size = UDim2.new(0, 200, 0, 30)
    dropdownFrame.Position = UDim2.new(0, 10, 0, 50)
    dropdownFrame.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    dropdownFrame.BorderSizePixel = 1

    local dropdownLabel = Instance.new("TextLabel", dropdownFrame)
    dropdownLabel.Text = label
    dropdownLabel.Size = UDim2.new(1, 0, 0.5, 0)
    dropdownLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    dropdownLabel.BackgroundTransparency = 1

    local dropdownButton = Instance.new("TextButton", dropdownFrame)
    dropdownButton.Size = UDim2.new(1, 0, 0.5, 0)
    dropdownButton.Position = UDim2.new(0, 0, 0.5, 0)
    dropdownButton.Text = "Select"
    dropdownButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    dropdownButton.BackgroundColor3 = Color3.fromRGB(100, 100, 100)

    local optionsFrame = Instance.new("Frame", parent)
    optionsFrame.Size = UDim2.new(0, 200, 0, 0)
    optionsFrame.Position = UDim2.new(0, 10, 0, 80)
    optionsFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    optionsFrame.BorderSizePixel = 1
    optionsFrame.Visible = false

    local options = {}

    local function addOption(text)
        local optionButton = Instance.new("TextButton", optionsFrame)
        optionButton.Size = UDim2.new(1, 0, 0, 30)
        optionButton.Text = text
        optionButton.TextColor3 = Color3.fromRGB(255, 255, 255)
        optionButton.BackgroundColor3 = Color3.fromRGB(80, 80, 80)

        optionButton.MouseButton1Click:Connect(function()
            callback(text)
            dropdownButton.Text = text
            optionsFrame.Visible = false
        end)

        table.insert(options, optionButton)
    end

    dropdownButton.MouseButton1Click:Connect(function()
        optionsFrame.Visible = not optionsFrame.Visible
        optionsFrame.Size = UDim2.new(0, 200, 0, #options * 30)
    end)

    return {
        Add = addOption,
        Clear = function()
            for _, btn in pairs(options) do
                btn:Destroy()
            end
            options = {}
            optionsFrame.Size = UDim2.new(0, 200, 0, 0)
            optionsFrame.Visible = false
        end,
        AddOption = addOption,
        SetCallback = callback
    }
end

local PetDropdown = createDropdown(KillTab, "Select Pet", function(text)
    -- Unequip all pets
    local petsFolder = LocalPlayer:WaitForChild("petsFolder")
    for _, folder in pairs(petsFolder:GetChildren()) do
        if folder:IsA("Folder") then
            for _, pet in pairs(folder:GetChildren()) do
                game:GetService("ReplicatedStorage").rEvents.equipPetEvent:FireServer("unequipPet", pet)
            end
        end
    end
    task.wait(0.2)
    local petName = text
    local petsToEquip = {}
    for _, pet in pairs(petsFolder:FindFirstChild("Unique"):GetChildren()) do
        if pet.Name == petName then
            table.insert(petsToEquip, pet)
        end
    end
    local maxPets = 8
    local equippedCount = math.min(#petsToEquip, maxPets)
    for i = 1, equippedCount do
        game:GetService("ReplicatedStorage").rEvents.equipPetEvent:FireServer("equipPet", petsToEquip[i])
        task.wait(0.1)
    end
end)

local wildWizardOption = PetDropdown:Add("Wild Wizard")
local mightyMonsterOption = PetDropdown:Add("Mighty Monster")

-- Auto Good Karma
KillTab:AddSwitch("Auto Good Karma", function(enabled)
    autoGoodKarma = enabled
    task.spawn(function()
        while autoGoodKarma do
            local character = LocalPlayer.Character
            local rightHand = character and character:FindFirstChild("RightHand")
            local leftHand = character and character:FindFirstChild("LeftHand")
            if character and rightHand and leftHand then
                for _, target in ipairs(Players:GetPlayers()) do
                    if target ~= LocalPlayer then
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
KillTab:AddSwitch("Auto Bad Karma", function(enabled)
    autoBadKarma = enabled
    task.spawn(function()
        while autoBadKarma do
            local character = LocalPlayer.Character
            local rightHand = character and character:FindFirstChild("RightHand")
            local leftHand = character and character:FindFirstChild("LeftHand")
            if character and rightHand and leftHand then
                for _, target in ipairs(Players:GetPlayers()) do
                    if target ~= LocalPlayer then
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

-- Auto Whitelist Friends
local friendWhitelistActive = false
KillTab:AddSwitch("Auto Whitelist Friends", function(state)
    friendWhitelistActive = state
    if state then
        for _, player in ipairs(Players:GetPlayers()) do
            if player ~= LocalPlayer and LocalPlayer:IsFriendsWith(player.UserId) then
                playerWhitelist[player.Name] = true
            end
        end
        Players.PlayerAdded:Connect(function(player)
            if friendWhitelistActive and player ~= LocalPlayer and LocalPlayer:IsFriendsWith(player.UserId) then
                playerWhitelist[player.Name] = true
            end
        end)
    else
        for name in pairs(playerWhitelist) do
            local friend = Players:FindFirstChild(name)
            if friend and LocalPlayer:IsFriendsWith(friend.UserId) then
                playerWhitelist[name] = nil
            end
        end
    end
end)

-- Whitelist / UnWhitelist TextBoxes
KillTab:AddTextBox("Whitelist", function(text)
    local target = Players:FindFirstChild(text)
    if target then
        playerWhitelist[target.Name] = true
    end
end)

KillTab:AddTextBox("UnWhitelist", function(text)
    local target = Players:FindFirstChild(text)
    if target then
        playerWhitelist[target.Name] = nil
    end
end)

-- Auto Kill
KillTab:AddSwitch("Auto Kill", function(enabled)
    autoKill = enabled
    task.spawn(function()
        while autoKill do
            local character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
            local rightHand = character:FindFirstChild("RightHand")
            local leftHand = character:FindFirstChild("LeftHand")
            local punch = LocalPlayer.Backpack:FindFirstChild("Punch")
            if punch and not character:FindFirstChild("Punch") then
                punch.Parent = character
            end
            if rightHand and leftHand then
                for _, target in ipairs(Players:GetPlayers()) do
                    if target ~= LocalPlayer and not playerWhitelist[target.Name] then
                        local targetChar = target.Character
                        local rootPart = targetChar and targetChar:FindFirstChild("HumanoidRootPart")
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

-- Target Selection Dropdown
local targetPlayerNames = {}
local selectedTarget = nil

local targetDropdown = createDropdown(KillTab, "Select Target", function(displayName)
    for _, player in ipairs(Players:GetPlayers()) do
        if player.DisplayName == displayName then
            if not table.find(targetPlayerNames, player.Name) then
                table.insert(targetPlayerNames, player.Name)
            end
            selectedTarget = player.Name
            break
        end
    end
end)

-- Remove selected target button
local removeTargetButton = Instance.new("TextButton", KillTab)
removeTargetButton.Size = UDim2.new(0, 200, 0, 30)
removeTargetButton.Position = UDim2.new(0, 10, 0, 250)
removeTargetButton.Text = "Remove Selected Target"
removeTargetButton.TextColor3 = Color3.fromRGB(255,255,255)
removeTargetButton.BackgroundColor3 = Color3.fromRGB(100, 0, 0)
removeTargetButton.MouseButton1Click:Connect(function()
    if selectedTarget then
        for i, v in ipairs(targetPlayerNames) do
            if v == selectedTarget then
                table.remove(targetPlayerNames, i)
                break
            end
        end
        selectedTarget = nil
    end
end)

-- Initialize dropdown with current players
for _, player in ipairs(Players:GetPlayers()) do
    if player ~= LocalPlayer then
        targetDropdown:Add(player.DisplayName)
        targetDropdownItems[player.Name] = player.DisplayName
    end
end

-- Player joins
Players.PlayerAdded:Connect(function(player)
    if player ~= LocalPlayer then
        targetDropdown:Add(player.DisplayName)
        targetDropdownItems[player.Name] = player.DisplayName
    end
end)

-- Player leaves
Players.PlayerRemoving:Connect(function(player)
    if targetDropdownItems[player.Name] then
        targetDropdownItems[player.Name] = nil
        -- Rebuild dropdown
        targetDropdown:Clear()
        for _, displayName in pairs(targetDropdownItems) do
            targetDropdown:Add(displayName)
        end
    end
    -- Remove from internal target list
    for i = #targetPlayerNames, 1, -1 do
        if targetPlayerNames[i] == player.Name then
            table.remove(targetPlayerNames, i)
        end
    end
end)

-- Start kill target switch
local killSwitch = false
KillTab:AddSwitch("Start Kill Target", function(state)
    killTarget = state
    task.spawn(function()
        while killTarget do
            local character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
            local rightHand = character:FindFirstChild("RightHand")
            local leftHand = character:FindFirstChild("LeftHand")
            local punch = LocalPlayer.Backpack:FindFirstChild("Punch")
            if punch and not character:FindFirstChild("Punch") then
                punch.Parent = character
            end

            if rightHand and leftHand then
                for _, name in ipairs(targetPlayerNames) do
                    local target = Players:FindFirstChild(name)
                    if target and target ~= LocalPlayer and target.Character then
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

-- View Player Dropdown
local spyTargetPlayerName = nil
local spyTargetDropdownItems = {}

local viewDropdown = createDropdown(KillTab, "Select View Target", function(displayName)
    for _, player in ipairs(Players:GetPlayers()) do
        if player.DisplayName == displayName then
            spyTargetPlayerName = player.Name
            break
        end
    end
end)

-- Fill with current players
for _, player in ipairs(Players:GetPlayers()) do
    if player ~= LocalPlayer then
        viewDropdown:Add(player.DisplayName)
        spyTargetDropdownItems[player.Name] = player.DisplayName
    end
end

-- Player joins
Players.PlayerAdded:Connect(function(player)
    if player ~= LocalPlayer then
        viewDropdown:Add(player.DisplayName)
        spyTargetDropdownItems[player.Name] = player.DisplayName
    end
end)

-- Player leaves
Players.PlayerRemoving:Connect(function(player)
    if spyTargetDropdownItems[player.Name] then
        spyTargetDropdownItems[player.Name] = nil
        -- Rebuild dropdown
        viewDropdown:Clear()
        for _, displayName in pairs(spyTargetDropdownItems) do
            viewDropdown:Add(displayName)
        end
    end
end)

-- Spying toggle
local spyingActive = false
KillTab:AddSwitch("View Player", function(enabled)
    spyingActive = enabled
    if not spyingActive then
        local cam = Workspace.CurrentCamera
        cam.CameraSubject = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") or LocalPlayer
        return
    end
    task.spawn(function()
        while spyingActive do
            local target = Players:FindFirstChild(spyTargetPlayerName)
            if target and target ~= LocalPlayer then
                local humanoid = target.Character and target.Character:FindFirstChild("Humanoid")
                if humanoid then
                    Workspace.CurrentCamera.CameraSubject = humanoid
                end
            end
            task.wait(0.1)
        end
    end)
end)

-- Remove Punch Anim Button
local removePunchAnimButton = Instance.new("TextButton", KillTab)
removePunchAnimButton.Size = UDim2.new(0, 200, 0, 30)
removePunchAnimButton.Position = UDim2.new(0, 10, 0, 300)
removePunchAnimButton.Text = "Remove Punch Anim"
removePunchAnimButton.TextColor3 = Color3.fromRGB(255,255,255)
removePunchAnimButton.BackgroundColor3 = Color3.fromRGB(150, 0, 0)
removePunchAnimButton.MouseButton1Click:Connect(function()
    -- Function to block specific animations
    local blockedAnimations = {
        ["rbxassetid://3638729053"] = true,
        ["rbxassetid://3638767427"] = true,
    }

    local function setupAnimationBlocking()
        local char = LocalPlayer.Character
        if not char or not char:FindFirstChild("Humanoid") then return end
        local humanoid = char:FindFirstChild("Humanoid")
        for _, track in pairs(humanoid:GetPlayingAnimationTracks()) do
            if track.Animation then
                local animId = track.Animation.AnimationId
                local animName = track.Name:lower()
                if blockedAnimations[animId] or animName:match("punch") or animName:match("attack") or animName:match("right") then
                    track:Stop()
                end
            end
        end
        -- Connect to animation played
        if not _G.AnimBlockConnection then
            _G.AnimBlockConnection = humanoid.AnimationPlayed:Connect(function(track)
                if track.Animation then
                    local animId = track.Animation.AnimationId
                    local animName = track.Name:lower()
                    if blockedAnimations[animId] or animName:match("punch") or animName:match("attack") or animName:match("right") then
                        track:Stop()
                    end
                end
            end)
        end
    end
    setupAnimationBlocking()

    -- Override tool activation for punch tools
    local function overrideToolActivation()
        local function processTool(tool)
            if tool and (tool.Name == "Punch" or tool.Name:match("Attack") or tool.Name:match("Right")) then
                if not tool:GetAttribute("ActivatedOverride") then
                    tool:SetAttribute("ActivatedOverride", true)
                    local connection = tool.Activated:Connect(function()
                        task.wait(0.05)
                        local char = LocalPlayer.Character
                        if char and char:FindFirstChild("Humanoid") then
                            for _, track in pairs(char.Humanoid:GetPlayingAnimationTracks()) do
                                if track.Animation then
                                    local animId = track.Animation.AnimationId
                                    local animName = track.Name:lower()
                                    if blockedAnimations[animId] or animName:match("punch") or animName:match("attack") or animName:match("right") then
                                        track:Stop()
                                    end
                                end
                            end
                        end
                    end)
                    if not _G.ToolConnections then _G.ToolConnections = {} end
                    _G.ToolConnections[tool] = connection
                end
            end
        end
        -- Process tools in backpack
        for _, tool in pairs(LocalPlayer.Backpack:GetChildren()) do
            processTool(tool)
        end
        -- Process tools in character
        local character = LocalPlayer.Character
        if character then
            for _, tool in pairs(character:GetChildren()) do
                if tool:IsA("Tool") then
                    processTool(tool)
                end
            end
        end
        -- Connect to new tools
        if not _G.BackpackAddedConnection then
            _G.BackpackAddedConnection = LocalPlayer.Backpack.ChildAdded:Connect(function(child)
                if child:IsA("Tool") then
                    task.wait(0.1)
                    processTool(child)
                end
            end)
        end
        if not _G.CharacterToolAddedConnection and character then
            _G.CharacterToolAddedConnection = character.ChildAdded:Connect(function(child)
                if child:IsA("Tool") then
                    task.wait(0.1)
                    processTool(child)
                end
            end)
        end
    end
    overrideToolActivation()

    -- Heartbeat monitoring to stop blocked animations
    if not _G.AnimMonitorConnection then
        _G.AnimMonitorConnection = game:GetService("RunService").Heartbeat:Connect(function()
            if tick() % 0.5 < 0.01 then
                local char = LocalPlayer.Character
                if char and char:FindFirstChild("Humanoid") then
                    for _, track in pairs(char.Humanoid:GetPlayingAnimationTracks()) do
                        if track.Animation then
                            local animId = track.Animation.AnimationId
                            local animName = track.Name:lower()
                            if blockedAnimations[animId] or animName:match("punch") or animName:match("attack") or animName:match("right") then
                                track:Stop()
                            end
                        end
                    end
                end
            end
        end)
    end
    -- Reconnect on character respawn
    if not _G.CharacterAddedConnection then
        _G.CharacterAddedConnection = LocalPlayer.CharacterAdded:Connect(function(newChar)
            task.wait(1)
            setupAnimationBlocking()
            overrideToolActivation()
            if _G.CharacterToolAddedConnection then _G.CharacterToolAddedConnection:Disconnect() end
            _G.CharacterToolAddedConnection = newChar.ChildAdded:Connect(function(child)
                if child:IsA("Tool") then
                    task.wait(0.1)
                    processTool(child)
                end
            end)
        end)
    end
end)

-- Function to recover punch animations
local function RecoveryPunch()
    if _G.AnimBlockConnection then _G.AnimBlockConnection:Disconnect() _G.AnimBlockConnection = nil end
    if _G.AnimMonitorConnection then _G.AnimMonitorConnection:Disconnect() _G.AnimMonitorConnection = nil end
    if _G.ToolConnections then
        for _, conn in pairs(_G.ToolConnections) do if conn then conn:Disconnect() end end
        _G.ToolConnections = nil
    end
    if _G.BackpackAddedConnection then _G.BackpackAddedConnection:Disconnect() _G.BackpackAddedConnection = nil end
    if _G.CharacterToolAddedConnection then _G.CharacterToolAddedConnection:Disconnect() _G.CharacterToolAddedConnection = nil end
    if _G.CharacterAddedConnection then _G.CharacterAddedConnection:Disconnect() _G.CharacterAddedConnection = nil end
end

local recoverPunchButton = Instance.new("TextButton", KillTab)
recoverPunchButton.Size = UDim2.new(0, 200, 0, 30)
recoverPunchButton.Position = UDim2.new(0, 10, 0, 340)
recoverPunchButton.Text = "Recover Punch Anim"
recoverPunchButton.TextColor3 = Color3.fromRGB(255,255,255)
recoverPunchButton.BackgroundColor3 = Color3.fromRGB(0, 150, 0)
recoverPunchButton.MouseButton1Click:Connect(function()
    RecoveryPunch()
end)

-- Auto Equip Punch
KillTab:AddSwitch("Auto Equip Punch", function(state)
    autoEquipPunch = state
    task.spawn(function()
        while autoEquipPunch do
            local punch = LocalPlayer.Backpack:FindFirstChild("Punch")
            if punch then
                punch.Parent = LocalPlayer.Character
            end
            task.wait(0.1)
        end
    end)
end)

-- Auto Punch without animation
KillTab:AddSwitch("Auto Punch [No Animation]", function(state)
    autoPunchNoAnim = state
    task.spawn(function()
        while autoPunchNoAnim do
            local punch = LocalPlayer.Backpack:FindFirstChild("Punch") or LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Punch")
            if punch then
                if punch.Parent ~= LocalPlayer.Character then
                    punch.Parent = LocalPlayer.Character
                end
                -- Fire punch without animation
                LocalPlayer.muscleEvent:FireServer("punch", "rightHand")
                LocalPlayer.muscleEvent:FireServer("punch", "leftHand")
            else
                autoPunchNoAnim = false
            end
            task.wait(0.01)
        end
    end)
end)

-- Auto Punch (with animation)
KillTab:AddSwitch("Auto Punch", function(state)
    _G.fastHitActive = state
    if state then
        task.spawn(function()
            while _G.fastHitActive do
                local punch = LocalPlayer.Backpack:FindFirstChild("Punch")
                if punch then
                    punch.Parent = LocalPlayer.Character
                    if punch:FindFirstChild("attackTime") then
                        punch.attackTime.Value = 0
                    end
                end
                task.wait(0.1)
            end
        end)
        task.spawn(function()
            while _G.fastHitActive do
                local punch = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Punch")
                if punch then
                    punch:Activate()
                end
                task.wait(0.1)
            end
        end)
    else
        local punch = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Punch")
        if punch then
            punch.Parent = LocalPlayer.Backpack
        end
    end
end)

-- Fast Punch
KillTab:AddSwitch("Fast Punch", function(state)
    _G.autoPunchActive = state
    if state then
        task.spawn(function()
            while _G.autoPunchActive do
                local punch = LocalPlayer.Backpack:FindFirstChild("Punch")
                if punch then
                    punch.Parent = LocalPlayer.Character
                    if punch:FindFirstChild("attackTime") then
                        punch.attackTime.Value = 0
                    end
                end
                task.wait()
            end
        end)
        task.spawn(function()
            while _G.autoPunchActive do
                local punch = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Punch")
                if punch then
                    punch:Activate()
                end
                task.wait()
            end
        end)
    else
        local punch = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Punch")
        if punch then
            punch.Parent = LocalPlayer.Backpack
        end
    end
end)

-- God Mode Switch
local godModeToggle = false
KillTab:AddSwitch("God Mode", function(state)
    godModeToggle = state
    if state then
        task.spawn(function()
            while godModeToggle do
                game:GetService("ReplicatedStorage").rEvents.brawlEvent:FireServer("joinBrawl")
                task.wait()
            end
        end)
    end
end)

-- Teleport / Follow System
local following = false
local followTarget = nil

local function followPlayer(targetPlayer)
    local myChar = LocalPlayer.Character
    local targetChar = targetPlayer.Character
    if not (myChar and targetChar) then return end
    local myHRP = myChar:FindFirstChild("HumanoidRootPart")
    local targetHRP = targetChar:FindFirstChild("HumanoidRootPart")
    if myHRP and targetHRP then
        local followPos = targetHRP.Position - (targetHRP.CFrame.LookVector * 3)
        myHRP.CFrame = CFrame.new(followPos, targetHRP.Position)
    end
end

local followDropdown = createDropdown(KillTab, "Teleport Player", function(selectedDisplayName)
    if selectedDisplayName and selectedDisplayName ~= "" then
        for _, plr in ipairs(Players:GetPlayers()) do
            if plr.DisplayName == selectedDisplayName then
                followTarget = plr
                following = true
                print("✅ Started following:", plr.Name)
                followPlayer(plr)
                break
            end
        end
    end
end)

-- Fill dropdown
for _, player in ipairs(Players:GetPlayers()) do
    if player ~= LocalPlayer then
        followDropdown:Add(player.DisplayName)
    end
end

-- Update list on join/leave
Players.PlayerAdded:Connect(function(player)
    if player ~= LocalPlayer then
        followDropdown:Add(player.DisplayName)
    end
end)

Players.PlayerRemoving:Connect(function(player)
    -- Refresh list
    followDropdown:Clear()
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer then
            followDropdown:Add(plr.DisplayName)
        end
    end
    -- Stop following if target left
    if followTarget and followTarget.Name == player.Name then
        followTarget = nil
        following = false
    end
end)

-- Button to stop following
local stopFollowBtn = Instance.new("TextButton", KillTab)
stopFollowBtn.Size = UDim2.new(0, 200, 0, 30)
stopFollowBtn.Position = UDim2.new(0, 10, 0, 380)
stopFollowBtn.Text = "Stop Following"
stopFollowBtn.TextColor3 = Color3.fromRGB(255,255,255)
stopFollowBtn.BackgroundColor3 = Color3.fromRGB(150, 0, 0)
stopFollowBtn.MouseButton1Click:Connect(function()
    following = false
    followTarget = nil
    print("⛔ Stopped following")
end)

-- Follow loop
task.spawn(function()
    while true do
        task.wait(0.01)
        if following and followTarget then
            local target = Players:FindFirstChild(followTarget.Name)
            if target then
                followPlayer(target)
            else
                following = false
                followTarget = nil
            end
        end
    end
end)

-- Re-follow on respawn
LocalPlayer.CharacterAdded:Connect(function()
    task.wait(1)
    if following and followTarget then
        local target = Players:FindFirstChild(followTarget.Name)
        if target then
            followPlayer(target)
        end
    end
end)

-- Auto Slam / Ground Slam
local autoSlam = false
KillTab:AddSwitch("Auto Slam", function(state)
    autoSlam = state
    if state then
        task.spawn(function()
            while autoSlam do
                local player = LocalPlayer
                local groundSlam = player.Backpack:FindFirstChild("Ground Slam") or (player.Character and player.Character:FindFirstChild("Ground Slam"))
                if groundSlam then
                    if groundSlam.Parent == player.Backpack then
                        groundSlam.Parent = player.Character
                    end
                    if groundSlam:FindFirstChild("attackTime") then
                        groundSlam.attackTime.Value = 0
                    end
                    player.muscleEvent:FireServer("slam")
                    groundSlam:Activate()
                end
                task.wait(0.1)
            end
        end)
    end
end)

-- "NaN" Button (change size to NaN)
local nanButton = Instance.new("TextButton", KillTab)
nanButton.Size = UDim2.new(0, 200, 0, 30)
nanButton.Position = UDim2.new(0, 10, 0, 420)
nanButton.Text = "NaN Button"
nanButton.TextColor3 = Color3.fromRGB(255,255,255)
nanButton.BackgroundColor3 = Color3.fromRGB(0, 0, 150)
nanButton.MouseButton1Click:Connect(function()
    local args = {"changeSize", 0/0}
    game:GetService("ReplicatedStorage"):WaitForChild("rEvents"):WaitForChild("changeSpeedSizeRemote"):InvokeServer(unpack(args))
end)

-- Change Time (Lighting)
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

local timeDropdown = createDropdown(KillTab, "Change Time", function(selection)
    -- Reset settings
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
end)

-- Add options to dropdown
for _, option in ipairs(timeOptions) do
    timeDropdown:Add(option)
end
