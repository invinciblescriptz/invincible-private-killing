-- Create a UI window
local player = game.Players.LocalPlayer
local gui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
gui.Name = "MyFeatureUI"

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 650, 0, 500)
frame.Position = UDim2.new(0.5, -325, 0.5, -250)
frame.BackgroundColor3 = Color3.fromRGB(0, 10, 0)
frame.BorderSizePixel = 0

local title = Instance.new("TextLabel", frame)
title.Size = UDim2.new(1, 0, 0, 50)
title.BackgroundColor3 = Color3.fromRGB(0, 20, 0)
title.TextColor3 = Color3.new(1, 1, 1)
title.Text = "Invincible Private Killing"
title.Font = Enum.Font.SourceSansBold
title.TextSize = 20

local yOffset = 60

-- Toggle: Auto Punch (Fast)
local punchToggle = Instance.new("TextButton", frame)
punchToggle.Size = UDim2.new(0, 300, 0, 40)
punchToggle.Position = UDim2.new(0, 20, 0, yOffset)
punchToggle.Text = "Auto Punch (Fast): OFF"
punchToggle.BackgroundColor3 = Color3.fromRGB(0, 50, 0)
punchToggle.TextColor3 = Color3.new(1, 1, 1)

-- Toggle: Auto Kill
local killToggle = Instance.new("TextButton", frame)
killToggle.Size = UDim2.new(0, 300, 0, 40)
killToggle.Position = UDim2.new(0, 20, 0, yOffset + 50)
killToggle.Text = "Auto Kill: OFF"
killToggle.BackgroundColor3 = Color3.fromRGB(0, 50, 0)
killToggle.TextColor3 = Color3.new(1, 1, 1)

-- Toggle: Auto Good Karma
local goodKarmaToggle = Instance.new("TextButton", frame)
goodKarmaToggle.Size = UDim2.new(0, 300, 0, 40)
goodKarmaToggle.Position = UDim2.new(0, 20, 0, yOffset + 100)
goodKarmaToggle.Text = "Auto Good Karma: OFF"
goodKarmaToggle.BackgroundColor3 = Color3.fromRGB(0, 50, 0)
goodKarmaToggle.TextColor3 = Color3.new(1, 1, 1)

-- Toggle: Auto Bad Karma
local badKarmaToggle = Instance.new("TextButton", frame)
badKarmaToggle.Size = UDim2.new(0, 300, 0, 40)
badKarmaToggle.Position = UDim2.new(0, 20, 0, yOffset + 150)
badKarmaToggle.Text = "Auto Bad Karma: OFF"
badKarmaToggle.BackgroundColor3 = Color3.fromRGB(0, 50, 0)
badKarmaToggle.TextColor3 = Color3.new(1, 1, 1)

-- Button: Auto Lift Gamepass
local liftGPButton = Instance.new("TextButton", frame)
liftGPButton.Size = UDim2.new(0, 300, 0, 40)
liftGPButton.Position = UDim2.new(0, 20, 0, yOffset + 200)
liftGPButton.Text = "Auto Lift Gamepass"
liftGPButton.BackgroundColor3 = Color3.fromRGB(0, 50, 0)
liftGPButton.TextColor3 = Color3.new(1,1,1)

-- Input: Enter Player Name for Follow
local followInput = Instance.new("TextBox", frame)
followInput.Size = UDim2.new(0, 250, 0, 40)
followInput.Position = UDim2.new(0, 350, 0, yOffset + 50)
followInput.PlaceholderText = "Enter Player Name"
followInput.Text = ""
followInput.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
followInput.TextColor3 = Color3.new(1,1,1)

-- Button: Follow Player
local followBtn = Instance.new("TextButton", frame)
followBtn.Size = UDim2.new(0, 250, 0, 40)
followBtn.Position = UDim2.new(0, 350, 0, yOffset + 100)
followBtn.Text = "Follow Player"
followBtn.BackgroundColor3 = Color3.fromRGB(0, 50, 0)
followBtn.TextColor3 = Color3.new(1,1,1)

-- Button: Stop Following
local stopFollowBtn = Instance.new("TextButton", frame)
stopFollowBtn.Size = UDim2.new(0, 250, 0, 40)
stopFollowBtn.Position = UDim2.new(0, 350, 0, yOffset + 150)
stopFollowBtn.Text = "Stop Following"
stopFollowBtn.BackgroundColor3 = Color3.fromRGB(0, 50, 0)
stopFollowBtn.TextColor3 = Color3.new(1,1,1)

-- Variables
local autoPunch = false
local autoKill = false
local autoGoodKarma = false
local autoBadKarma = false
local spying = false
local targetPlayerName = nil
local whitelist = {}

local Players = game:GetService("Players")
local LP = Players.LocalPlayer

-- Helper function to get HRP
local function getHRP(character)
    if character then
        return character:FindFirstChild("HumanoidRootPart")
    end
    return nil
end

-- Toggle handlers
punchToggle.MouseButton1Click:Connect(function()
    autoPunch = not autoPunch
    punchToggle.Text = "Auto Punch (Fast): " .. (autoPunch and "ON" or "OFF")
end)

killToggle.MouseButton1Click:Connect(function()
    autoKill = not autoKill
    killToggle.Text = "Auto Kill: " .. (autoKill and "ON" or "OFF")
end)

goodKarmaToggle.MouseButton1Click:Connect(function()
    autoGoodKarma = not autoGoodKarma
    goodKarmaToggle.Text = "Auto Good Karma: " .. (autoGoodKarma and "ON" or "OFF")
end)

badKarmaToggle.MouseButton1Click:Connect(function()
    autoBadKarma = not autoBadKarma
    badKarmaToggle.Text = "Auto Bad Karma: " .. (autoBadKarma and "ON" or "OFF")
end)

-- Auto Lift Gamepass button
liftGPButton.MouseButton1Click:Connect(function()
    pcall(function()
        local gpFolder = game.ReplicatedStorage:FindFirstChild("gamepassIds")
        if not gpFolder then warn("No gamepassIds folder") return end
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
        print("Gamepasses lifted")
    end)
end)

-- Follow Player
followBtn.MouseButton1Click:Connect(function()
    local name = followInput.Text
    if name ~= "" then
        targetPlayerName = name
        spying = true
        print("Following: " .. name)
    end
end)

-- Stop Following
stopFollowBtn.MouseButton1Click:Connect(function()
    spying = false
    workspace.CurrentCamera.CameraSubject = LP.Character and LP.Character:FindFirstChild("Humanoid") or LP
    print("Stopped following")
end)

-- Main loop
spawn(function()
    while true do
        wait(0.05)

        -- Auto Punch
        if autoPunch and _G.NexusRunning then
            pcall(function()
                if muscleEvent then
                    muscleEvent:FireServer("punch", "rightHand")
                    muscleEvent:FireServer("punch", "leftHand")
                end
                local char = LP.Character
                if char then
                    local punch = char:FindFirstChild("Punch") or LP.Backpack:FindFirstChild("Punch")
                    if punch and punch.Parent ~= char then
                        punch.Parent = char
                        punch:Activate()
                    elseif punch and punch.Parent == char then
                        local atk = punch:FindFirstChild("attackTime")
                        if atk and atk:IsA("NumberValue") then atk.Value = 0.01 end
                        punch:Activate()
                    end
                end
            end)
        end

        -- Auto Kill
        if autoKill then
            local myChar = LP.Character
            if myChar then
                local rightHand = myChar:FindFirstChild("RightHand")
                local leftHand = myChar:FindFirstChild("LeftHand")
                local punch = LP.Backpack:FindFirstChild("Punch")
                if punch and not myChar:FindFirstChild("Punch") then
                    punch.Parent = myChar
                end
                if rightHand and leftHand then
                    for _, target in pairs(Players:GetPlayers()) do
                        if target ~= LP and not whitelist[target.Name] then
                            local tChar = target.Character
                            if tChar then
                                local root = tChar:FindFirstChild("HumanoidRootPart")
                                local hum = tChar:FindFirstChild("Humanoid")
                                if root and hum and hum.Health > 0 then
                                    pcall(function()
                                        firetouchinterest(rightHand, root, 1)
                                        firetouchinterest(leftHand, root, 1)
                                        firetouchinterest(rightHand, root, 0)
                                        firetouchinterest(leftHand, root, 0)
                                    end)
                                end
                            end
                        end
                    end
                end
            end
        end

        -- Follow target
        if spying and targetPlayerName then
            local target = Players:FindFirstChild(targetPlayerName)
            if target and target.Character then
                local targetHRP = getHRP(target.Character)
                local myChar = LP.Character
                local myHRP = getHRP(myChar)
                if targetHRP and myHRP then
                    myHRP.CFrame = targetHRP.CFrame * CFrame.new(0, 0, -3)
                end
            end
        end

        -- Auto Karma logic (auto touch interactions)
        if autoGoodKarma then
            for _, target in pairs(Players:GetPlayers()) do
                if target ~= LP and target.Character then
                    local evilKarma = target.Character:FindFirstChild("evilKarma")
                    local goodKarma = target.Character:FindFirstChild("goodKarma")
                    if evilKarma and goodKarma then
                        if evilKarma.Value > goodKarma.Value then
                            local root = target.Character:FindFirstChild("HumanoidRootPart")
                            local rightHand = LP.Character and LP.Character:FindFirstChild("RightHand")
                            local leftHand = LP.Character and LP.Character:FindFirstChild("LeftHand")
                            if root and rightHand and leftHand then
                                firetouchinterest(rightHand, root, 1)
                                firetouchinterest(leftHand, root, 1)
                                firetouchinterest(rightHand, root, 0)
                                firetouchinterest(leftHand, root, 0)
                            end
                        end
                    end
                end
            end
        end

        if autoBadKarma then
            for _, target in pairs(Players:GetPlayers()) do
                if target ~= LP and target.Character then
                    local evilKarma = target.Character:FindFirstChild("evilKarma")
                    local goodKarma = target.Character:FindFirstChild("goodKarma")
                    if evilKarma and goodKarma then
                        if goodKarma.Value > evilKarma.Value then
                            local root = target.Character:FindFirstChild("HumanoidRootPart")
                            local rightHand = LP.Character and LP.Character:FindFirstChild("RightHand")
                            local leftHand = LP.Character and LP.Character:FindFirstChild("LeftHand")
                            if root and rightHand and leftHand then
                                firetouchinterest(rightHand, root, 1)
                                firetouchinterest(leftHand, root, 1)
                                firetouchinterest(rightHand, root, 0)
                                firetouchinterest(leftHand, root, 0)
                            end
                        end
                    end
                end
            end
        end
    end
end)

print("All features are active.")
