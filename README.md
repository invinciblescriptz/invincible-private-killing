-- Full feature-rich GUI with draggable functionality
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

-- Helper functions to create buttons and inputs
local function createButton(text, yPos)
    local btn = Instance.new("TextButton", frame)
    btn.Size = UDim2.new(0, 300, 0, 40)
    btn.Position = UDim2.new(0, 20, 0, yPos)
    btn.Text = text
    btn.BackgroundColor3 = Color3.fromRGB(0, 50, 0)
    btn.TextColor3 = Color3.new(1, 1, 1)
    return btn
end

local function createInput(placeholder, yPos)
    local input = Instance.new("TextBox", frame)
    input.Size = UDim2.new(0, 250, 0, 40)
    input.Position = UDim2.new(0, 350, 0, yPos)
    input.PlaceholderText = placeholder
    input.Text = ""
    input.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    input.TextColor3 = Color3.new(1, 1, 1)
    return input
end

-- Creating UI elements
local punchToggle = createButton("Auto Punch (Fast): OFF", yOffset)
local killToggle = createButton("Auto Kill: OFF", yOffset + 50)
local goodKarmaToggle = createButton("Auto Good Karma: OFF", yOffset + 100)
local badKarmaToggle = createButton("Auto Bad Karma: OFF", yOffset + 150)
local liftGPButton = createButton("Auto Lift Gamepass", yOffset + 200)

local followInput = createInput("Enter Player Name", yOffset + 50)
local followBtn = createButton("Follow Player", yOffset + 100)
local stopFollowBtn = createButton("Stop Following", yOffset + 150)

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
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

-- Helper function to get HumanoidRootPart
local function getHRP(character)
    if character then
        return character:FindFirstChild("HumanoidRootPart")
    end
    return nil
end

-- Button handlers
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

followBtn.MouseButton1Click:Connect(function()
    local name = followInput.Text
    if name ~= "" then
        targetPlayerName = name
        spying = true
        print("Following: " .. name)
    end
end)

stopFollowBtn.MouseButton1Click:Connect(function()
    spying = false
    workspace.CurrentCamera.CameraSubject = LP.Character and LP.Character:FindFirstChild("Humanoid") or LP
    print("Stopped following")
end)

-- Main feature loop
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
        -- Follow Player
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
        -- Auto Karma (Good)
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
        -- Auto Karma (Bad)
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

-- ==================== Make GUI draggable ====================
local dragging = false
local dragStart, startPos

title.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = frame.Position
        -- To capture mouse release
        local conn
        conn = UserInputService.InputEnded:Connect(function(input2)
            if input2.UserInputType == Enum.UserInputType.MouseButton1 then
                dragging = false
                conn:Disconnect()
            end
        end)
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
        local delta = input.Position - dragStart
        frame.Position = startPos + UDim2.new(0, delta.X, 0, delta.Y)
    end
end)
